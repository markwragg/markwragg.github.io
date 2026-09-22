---
title: Speed up Azure DevOps pipelines with the Cache task
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2026/rev-counter.jpg"
  teaser: "/content/images/2026/rev-counter.jpg"
date: '2026-09-20 18:00:00'
tags:
- azuredevops
- pipelines
- powershell
draft: true
---

I maintain a number of [PowerShell modules in GitHub](https://github.com/markwragg?tab=repositories&language=powershell), and use a relatively identical Azure DevOps pipeline to build, test and publish changes to the modules. While making some improvements to one of the modules recently, I noticed that an early step in the pipeline was taking a few minutes to run each time, and wondered if there might be a way to speed it up. I discovered Azure DevOps has a task called [Cache](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/cache-v2?view=azure-pipelines), which can be used for exactly this purpose. This blog post describes how my PowerShell build pipeline works, and how the Cache task reduced the time it took to run one of the tasks from around 3 minutes, to a few seconds.

### About pipeline caching

You can learn more about [how pipeline caching works](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/caching?view=azure-devops&tabs=bundler#cache-task-:-how-it-works) via Microsoft Learn. The underlying architecture includes:
- The control plane: a high-throughput key-value storage that indexes the dynamic cache keys against branch permissions and access timestamps
- The data plane (Content Addressable Storage): a deduplication engine that ensures identical files are only stored once
- The physical backend: A managed Azure Blog Storage pool, deployed in the same geographic region as your Azure DevOps organisation to ensure maximum backbone network speeds.

When you add the Cache task to a pipeline, two tasks execute during the runtime:

1. The task executes at the point you added it to the pipeline. It checks if the file/s you have requested exist in the cache by creating a hash of their contents and sending it to the cache. If there is a cache hit, the task connects to Azure Storage and downloads the files. On a miss the task immediately terminates.
2. At the end of your pipeline execution, a Post-job Cache task executes (if the main pipeline steps have succeeded, to avoid caching broken files). If there was a cache hit in the earlier task, the post-job terminates as the file/s are already cached. If there was a miss, the pipeline compresses the files and streams them to Azure storage.

Files in the cache automatically expire after 7 days, but every time you use the files in the cache this clock is reset. It is also worth noting that caches are scoped by Project, Pipeline and Branch, and there is a hierarchy. A feature branch can read/restore caches from a parent branch (e.g main) but it can only upload/overwrite for a cache key that belongs to its own branch. This prevents a rogue PR from potentially poisoning the cache.

### My pipeline

My PowerShell pipeline was originally executed via a CI/CD tool called AppVeyor, and was borrowed from other members of the PowerShell community (I think predominantly [Warren Frame](https://github.com/RamblingCookieMonster), aka [PSCookieMonster](https://bsky.app/profile/pscookiemonster.bsky.social)). I decided some years ago to port the pipeline to Azure DevOps, partly as a learning exercise as it was becoming increasingly common in my day to day work. The structure of the pipeline didn't change much (and still hasn't), and works as follow:

- The pipeline executes automatically for any changes to the master branch, or for any PRs that would merge into master.
- Installs the [PSDepend](https://github.com/PowerShellOrg/PSDepend) PowerShell module, which is a PowerShell dependency handler. This takes a simple psd1 file as input, which contains a list of modules that the build pipeline depends on for other tasks, and installs those modules if they aren't already present. The modules my build pipeline depends on include:
  - BuildHelpers
  - Pester
  - PlatyPS
  - PSake
  - PSDeploy
  - PSScriptAnalyzer
- PSDepend installs specific versions of these modules, to ensure my builds continue to run as expected even when these modules are changed / updated.
- Using tasks grouped by PSake, the rest of the build pipeline then executes the following stages:
  - Builds the PowerShell module into a single psm1 file. In the repository, each cmdlet / function is maintained as separate files, but this impacts performance when Importing the module, which is resolved by publishing the module as a single file.
  - Runs PSScriptAnalyzer to ensure the module implements good PowerShell practices
  - Executes the Pester unit tests to ensure the latest changes have not broken any existing functionality and work as expected. This task also calculates code coverage, and updates the Readme.md with the resulting coverage percentage as a badge.
  - Exports the PowerShell help output for each public cmdlet in the module into a markdown file, that is then later published back into the repo.
  - If a deployment is required (which is determined by whether or not a `# !Deploy` section has been added to the changelog.md, as I previously described in this blog post on [changelog driven deployments](https://wragg.io/changelog-driven-deployments/)), and the build is executing against Master, a task runs to publish the latest version of the module in the PowerShell Gallery.
  - Publishes the updated Documentation and Readme markdown files back to the GitHub repo.
  - Publishes the results of the Pester tests into the Test area of the Azure DevOps build.
  - Publishes the PowerShell module as an output artifact of the Azure DevOps build (where it could then be consumed directly by other Azure DevOps pipelines if desired).

  ### The performance issue

  The two longest running tasks in the pipeline were the Install Dependencies task, which as described above, needed to install specific versions of 6 different PowerShell modules, and the Test task, which executes the Pester tests (which varies for each module depending on how many test there are to execute).

  ### The cache task solution

