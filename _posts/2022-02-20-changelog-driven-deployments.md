---
title: Changelog Driven Deployments
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2022/02/change-med.jpg"
  teaser: "/content/images/2022/02/change-sm.jpg"
date: '2022-02-20 14:00:00'
tags:
- powershell
- azuredevops
- cicd
- github
---

A [changelog](https://keepachangelog.com/en/1.0.0/) is a useful addition to any project, as it provides users and contributors with a summary of notable changes between each release. One way to ensure you always update your changelog as part of any new release is by making it part of the the automated deployment process. This blog post describes how I've implemented that for the PowerShell modules I maintain in GitHub.

My PowerShell modules are built and deployed using scripts that perform the following tasks whenever a PR or commit is made to the Master branch:

> Note: The basis of these tasks/scripts have been lifted and modified from other PowerShell community members, but I can't recall who specifically to give them credit.

1. Combine the individual PowerShell functions into a single module file. This improves performance when the module is loaded.

2. Execute the Pester tests to validate the code is functioning correctly and to update the `README.md` with the current code coverage percentage.

3. Generate automatic PowerShell Help documentation for each of the module's commands into the /Documentation folder of the repo. This gives users the option to browse help online.

4. Deploy the module to the PowerShell Gallery, but only _if_ the branch is Master and the `CHANGELOG.md` file includes `## !Deploy`. This line of the ChangeLog is then modified during this task to include the version number of the module being deployed and the date of the deployment.

5. Finally the changes that have been made to the source files (the updates to the README.md, CHANGELOG.md and any changes under /Documentation) are committed back to source control.

You can look at these tasks in more detail by looking at the /Build folder under any of [my PowerShell projects in GitHub](https://github.com/markwragg/PowerShell-Influx/blob/master/Build).

Making the `CHANGELOG.md` part of the deployment logic is achieved by adding the following to the Deploy task:

```powershell
if (Get-Item "$ProjectRoot/CHANGELOG.md") {
        
    $ChangeLog = Get-Content "$ProjectRoot/CHANGELOG.md"

    if ($ChangeLog -contains '## !Deploy') {

        $Params = @{
            Path    = "$ProjectRoot/Build/deploy.psdeploy.ps1"
            Force   = $true
            Recurse = $false
        }

        Invoke-PSDeploy @Verbose @Params

        # Update ChangeLog with deployment version and date
        $ChangeLog = $ChangeLog -replace '## !Deploy', "## [$Version] - $(Get-Date -Format 'yyyy-MM-dd')"
        Set-Content -Path "$ProjectRoot/CHANGELOG.md" -Value $ChangeLog
    }
    else {
        Write-Host 'CHANGELOG.md did not contain ## !Deploy. Skipping deployment.'
    }
}
else {
    Write-Host "$ProjectRoot/CHANGELOG.md not found. Skipping deployment."
}
```

Granting Azure DevOps permissions to write to your code repositories can be done by using [GitHub App authentication](https://docs.microsoft.com/en-us/azure/devops/pipelines/repos/github?view=azure-devops&tabs=yaml#github-app-authentication). In my build pipeline committing the changes back to the repo is completed as follows:

```powershell
  Set-Location $ProjectRoot
  $Module = $env:BHProjectName

  git --version
  git config --global user.email "build@azuredevops.com"
  git config --global user.name "AzureDevOps"
  git checkout $env:BUILD_SOURCEBRANCHNAME
  git add Documentation/*.md
  git add README.md
  git add CHANGELOG.md
  git commit -m "[skip ci] AzureDevOps Build $($env:BUILD_BUILDID)"
  git push
```

It is important to include `[skip ci]` in the commit message otherwise you risk creating an infinite loop of builds and check ins. The `[skip ci]` tag is a built in way to tell Azure DevOps not to build something you've committed.

Here's an example of the ChangeLog being updated for a recent deployment of my PowerShell-Influx module:

![changelog deployment example commit](/content/images/2022/02/changelog-deploy-example.png)

Here's the CI pupeline updating the `CHANGELOG.md` to include the new version number and date as the title:

![changelog deployment example commit](/content/images/2022/02/changelog-deploy-example-2.png)

And here's how the final ChangeLog looks:

![changelog deployment example commit](/content/images/2022/02/changelog-deploy-example-3.png)

If you implement this technique its a good idea to also include a `CONTRIBUTING.md` file in your project to let people know that updating the ChangeLog is a mandatory part of the deployment process:

![changelog deployment example commit](/content/images/2022/02/contributing-example.png)

-- https://github.com/markwragg/PowerShell-Influx/blob/master/CONTRIBUTING.md