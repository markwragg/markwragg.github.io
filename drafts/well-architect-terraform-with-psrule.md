---
title: Well-architect your Azure Terraform with PSRule
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/one-ring.jpg"
  teaser: "/content/images/2025/one-ring.jpg"
date: '2025-02-08 09:00:00'
tags:
- azure
- psrule
- terraform
- azuredevops
- pipelines
- powershell
---

> "All that is gold does not glitter, not all those who wander are lost; the old that is strong does not wither, deep roots are not reached by the frost." — J.R.R Tolkien

Many DevOps tools were gifted to the Cloud Engineers, who above all else, desired automation. For within these tools was bound the strength and will to govern the Cloud.
But they were all of them deceived, for another tool was made.
In the land of Microsoft, in the fires of Mount Azure, the Architect [Bernie White](https://www.linkedin.com/in/bernie-white/) forged, in open source, a tool to validate all others.
And into this tool he poured his creativity, his mastery and his desire to improve your infrastructure. One tool to [PSRule](https://microsoft.github.io/PSRule/v2/) them all.

I suspect it was something like that anyway. PSrule is 

Fortunately [installing PSRule](https://azure.github.io/PSRule.Rules.Azure/install/) requires no difficult journey (or gathering of fellowships), although you may feel quite heroic once you do. Simply do the following:

- Install PowerShell 7 (if you haven't already)
- Install PSRule from the PowerShell Gallery via `Install-Module -Name 'PSRule.Rules.Azure' -Repository PSGallery -Scope CurrentUser`

For Azure Spring Clean 2024, [Dan Rios blogged about PSRule](https://rios.engineer/azure-spring-clean-azure-best-practice-for-bicep-with-psrule/)