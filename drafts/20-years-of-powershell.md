---
title: 20 Years of PowerShell
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2026/balloons.jpg"
  teaser: "/content/images/2026/balloons.jpg"
date: '2026-11-14 09:00:00'
tags:
- powershell
- career
- reflection
excerpt: "PowerShell 1.0 shipped on 14 November 2006. Twenty years on to the day, I look back at how it's shaped the way I work, the career it built, and the community around it."
---

<!--
SCAFFOLD NOTES (delete this comment block before publishing):
- Today, 14 Nov 2026, is exactly 20 years since PowerShell 1.0 shipped (14 Nov 2006). That's the hook.
- Sections marked with *(...)* are prompts for you to replace with your own story - don't leave them in.
- The version history table is factual and safe to keep/trim/correct, it's not part of the personal narrative.
- Pick/record a header image - something like an old terminal next to a modern one, or a "then vs now" screenshot of your own scripts.
-->

Twenty years ago today, on 14 November 2006, Microsoft shipped PowerShell 1.0 as a downloadable add-on for Windows XP and Server 2003. It wasn't bundled with the OS, it wasn't fast, and most of the Windows admin world was still living in `.bat` files and VBScript. Two decades later it's hard to imagine doing my job without it.

[Jeffrey Snover](https://en.wikipedia.org/wiki/Jeffrey_Snover) is rightly credited as the creator of PowerShell - he wrote the original "Monad Manifesto" back in 2002 that proposed an object-based shell for Windows, and spent years fighting internal resistance at Microsoft to get it built. But Snover himself has always been quick to deflect sole credit: *"People give me too much credit for PowerShell. PowerShell is great because we had a series of rockstar engineers add their awesomeness to the project,"* he's said, specifically calling out people like Bruce Payette (co-designer of the language itself) among the team who actually built it. I was reminded of this watching [the 20th anniversary celebration](https://www.youtube.com/watch?v=P_unk_59wWs) yesterday, where Snover was keen to credit Kenneth Hansen, PowerShell's long-time Lead Program Manager, who's been there since close to the start and co-hosted the 10th anniversary celebration with Snover back in 2016 too. It's a nice reminder that even the tools we think of as one person's vision are almost always a team effort underneath.

This post is a personal one. Less "here's a technique" and more a look back at how a scripting language ended up being the backbone of my career.

### Life before PowerShell

*(Your reflection here - e.g. what were you using day to day before PowerShell existed or before you adopted it: VBScript, batch files, cmd.exe, manual GUI work? What was your job at the time?)*

### How I found it

*(When did you first encounter PowerShell - v1, v2, v3? Was it a specific problem that pushed you to it, a colleague, a course? What was the "aha" moment that hooked you - pipelines, objects instead of text, remoting?)*

### A (very abridged) version history

A rough timeline of the releases that mark out the eras I've lived through with it - adjust/trim to the versions that actually meant something to you:

| Version | Shipped | Notable for me |
|---|---|---|
| 1.0 | Nov 2006 | ~129 cmdlets, the object pipeline, .NET integration - a real automation engine, not just a text shell |
| 2.0 | Oct 2009 (Win7 / Server 2008 R2) | Remoting, modules, ISE |
| 3.0 | Sep 2012 (Win8 / Server 2012) | Workflows, scheduled jobs |
| 4.0 | Oct 2013 (Win8.1 / Server 2012 R2) | Desired State Configuration |
| 5.0 / 5.1 | 2016 (Win10 / Server 2016) | Classes, PowerShellGet, the Gallery |
| 6.0 "Core" | Jan 2018 | Open source, cross-platform, .NET Core |
| 7.0 | Mar 2020 | .NET Core 3.1, Windows compatibility layer, ternary operator, `&&` / `\|\|` pipeline chains, `ForEach-Object -Parallel` |
| 7.1 | Jan 2021 | .NET 5, concise error view introduced (experimental) |
| 7.2 LTS | Nov 2021 | Concise error view on by default, `$PSStyle` for ANSI/colour output, Secrets management went GA |
| 7.3 | Nov 2022 | .NET 7, official Windows ARM64 support, native command argument passing improvements |
| 7.4 LTS | Nov 2023 | .NET 8, `$PSNativeCommandUseErrorActionPreference` - native command failures can now respect `$ErrorActionPreference` |
| 7.5 | Jan 2025 | .NET 9, array/pipeline cmdlet refinements |
| 7.6 LTS | Mar 2026 | .NET 10 LTS, replaced `BinaryFormatter` with a secure implementation (fixing `Out-GridView`), tab-completion fixes |
| 7.7 (STS) | Nov 2026 | .NET 11, security hardening (Constrained Language Mode, implicit remoting), `New-TemporaryDirectory`, time-sortable v7 GUIDs by default |

### The jobs it shaped

*(Walk through 2-3 roles/projects where PowerShell was the differentiator - the automate-yourself-a-promotion story, the AD/infra admin years, the move into DevOps/cloud. You've already written pieces of this - e.g. your [automate yourself a better job](/automate-yourself-a-better-job-with-powershell/) and [PowerShell millionaire](/how-i-became-a-powershell-millionaire/) posts have raw material to pull from and link back to.)*

### Publishing, community, and the Gallery

*(Your module history is already quantified in the millionaire post - pull a couple of the most meaningful numbers/stories forward here rather than repeating all of it. Also: MVP status if applicable, conferences, the PowerShell Podcast, user groups, Twitter/Bluesky - whatever built your sense of "this is a community, not just a tool".)*

### Open source and the cross-platform shift

*(Your take on the v6 "Core" rewrite - open-sourcing, Linux/Mac support, the DevOps/cloud era it enabled for you specifically - Azure, Terraform, pipelines.)*

### What changed in how I think

*(The bigger lesson - e.g. pipelines/objects changing how you reason about data, or automation-first thinking changing how you approach any repetitive task, scripting as a default reflex rather than a specialist skill.)*

### The next 20 years

*(Where you think it goes from here - AI-assisted scripting, agents calling PowerShell, your own recent experiments like the [awesome-powershell-modules](/building-a-self-maintaining-awesome-list-with-github-actions-and-claude/) project. Close on something personal rather than a prediction.)*

---

*(Closing line/image - your usual sign-off.)*
