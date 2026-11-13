---
title: Building an Awesome List for PowerShell Modules, with Help from AI
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2026/powershell-modules-banner.jpg"
  teaser: "/content/images/2026/powershell-modules-banner.jpg"
tags:
- powershell
- github-actions
- ai
- claude
- automation
draft: true
---

<!-- TODO: header image - no image currently saved under content/images/2026 for this post. -->


I recently created [awesome-powershell-modules](https://github.com/markwragg/awesome-powershell-modules), a curated ["awesome list"](https://github.com/sindresorhus/awesome) of PowerShell modules covering package management, testing, automation, security, cloud administration and more. This blog post covers why I built it, how the repo's GitHub Actions pipelines keep it from going stale, and how I've started using Claude to help review and triage new module submissions each week.

### Why another list?

- The PowerShell Gallery has thousands of modules of wildly varying quality/maintenance - hard to know what's worth reaching for first.
- Goal: a curated, opinionated, actively-maintained list rather than an exhaustive index.
- Link to the repo / README badges (Lint Check, Repo Check, Link Check) and the categorisation (20 categories from Package & Dependency Management through to Fun & Novelty).
- `CONTRIBUTING.md` sets the bar for entries: must be free/open source, actively maintained, documented, at least 30 days old, alphabetically ordered within its category, terse factual descriptions (no "A" / "An", no marketing language).

### Keeping the list honest with GitHub Actions

An awesome list rots quickly - links break, repos get archived, modules stop being maintained. Four workflows run on a schedule (and on push/PR) to catch this automatically:

- **`awesome-lint.yml`** - runs `npx awesome-lint` on push/PR to enforce the awesome-list conventions (formatting, alphabetical order, description style).
- **`link-check.yml`** - runs [lychee](https://github.com/lycheeverse/lychee-action) weekly against README.md to find dead links, with its own cache to avoid re-checking everything every run.
- **`archived-check.yml`** - runs `Test-ArchivedRepos.ps1` weekly, which extracts every `github.com/<owner>/<repo>` link from README.md and checks the GitHub API for `archived: true`. Has an `-ExcludeRepo` escape hatch for repos archived deliberately (e.g. `PowerShell/SecretManagement`, archived by the PowerShell team as "feature complete" rather than abandoned).
- **`stale-check.yml`** - runs `Test-StaleRepos.ps1` weekly, flagging any linked repo with no push in over 5 years (configurable), emitting `::warning::` annotations so they surface in the Actions UI.

Include a snippet of one of these scripts (`Test-ArchivedRepos.ps1` is the simplest) and a screenshot of the Actions tab / a workflow summary.

### Discovering new modules automatically

- `scripts/Find-NewGalleryModules.ps1` queries the PowerShell Gallery's OData feed directly (not `Find-Module`/PSResourceGet) for speed, looking for modules whose **first-ever** version was published 30-37 days ago.
- Why a window and not "brand new": gives a module a few weeks to prove it isn't abandoned on day two, while still catching it within the next weekly run.
- Explain the "broad net then confirm" approach: a cheap single filter query catches any version created in the window (so a module that's since shipped newer versions doesn't dodge the window), then a couple of small follow-up requests per *new* candidate confirm the module's earliest version actually falls inside it, and fetch its current stable release metadata.
- Already-listed modules are excluded first (by normalised name and by Gallery ID parsed out of existing README links), so that lookup cost is only paid for genuinely new candidates.
- Worth showing the OData `$filter` query as a code snippet - it's a nice example of talking to the Gallery without any module dependency at all.

### Letting Claude triage the candidates

This is the interesting bit. `new-modules-check.yml` runs weekly (and on `workflow_dispatch`):

1. Runs `Find-NewGalleryModules.ps1` and writes the results to the job summary as a markdown table *and* to `candidates.json`.
2. If there are candidates, calls [`anthropics/claude-code-action@v1`](https://github.com/anthropics/claude-code-action) with a scoped tool allow-list (`Read,Edit,WebFetch,Bash(git:*),Bash(gh:*),Bash(date:*)` - notably no unrestricted shell) and a prompt that:
   - Gives Claude the same inclusion criteria as human contributors get in `CONTRIBUTING.md` (actively maintained, single clear purpose, documented, not a near-duplicate of something already listed).
   - Asks it to look at each candidate's project/Gallery links (and source repo) and decide include/exclude with one concise reason each.
   - For includes: insert into the correct existing category, alphabetically, matching the exact `- [Name](url) - Description.` format and tone.
   - For excludes: don't touch the README at all.
   - If anything was added: create a branch named `new-modules/<date>`, commit, push, and open a PR with `gh pr create`, with a body listing every candidate under "### Added" / "### Not added" headings.
   - If nothing was a good fit: skip the PR entirely and just append the reasoning to the job summary - avoids opening noise PRs just to say "no".
3. Explicitly never pushes to `main` - the model's only path to changing the list is a PR, which I still review and merge myself.

### A real example

The first run opened [PR #1](https://github.com/markwragg/awesome-powershell-modules/pull/1) against ~51 candidates found in the Gallery:

- Added **PSZoxide** and **PWRSWriteColorEX**, each with a one-line justification.
- Excluded the rest with specific, varied reasoning - e.g. recognising a family of 9 `IdentityCommand.*` vendor SDK modules as "same family, too new to judge" rather than listing each individually, spotting `TerminalGlyphs` as a near-duplicate of the already-listed Terminal-Icons, and rejecting internal/plumbing modules (`NSP.RepoTools`, `Boyles.PowerShell.Core`) that aren't general-purpose.
- Worth pulling 4-5 of the more interesting exclusion reasons into the post verbatim as examples of the judgement calls it's making beyond simple keyword matching.
- Screenshot the PR / Actions run.

### Scripts for visibility into the list itself

- `Get-ModuleAuthor.ps1` / `Get-ModuleDetails.ps1` - not part of the pipeline, but handy for sanity-checking the list: who are the most prolific authors on it, what are the real download counts, etc. Maybe a table of top authors as of writing.

### Reflections

- Treat the AI like any other contributor: least-privilege tool access, output constrained to a PR for review, not direct write access to `main`.
- Giving it the *same written criteria* as human contributors (reusing `CONTRIBUTING.md` language in the prompt) kept its decisions consistent with how I'd have judged them myself.
- The weekly cadence means I'm no longer manually trawling new Gallery listings - I just review a PR (or nothing, most weeks).
- Caveats to mention: still reviewing every PR before merge, occasional judgement calls I might disagree with, cost of running the action.

### Conclusion

- Link back to the repo and README, and to `CONTRIBUTING.md` for anyone wanting to suggest a module (human-submitted PRs go through the same lint/link/archive checks).
