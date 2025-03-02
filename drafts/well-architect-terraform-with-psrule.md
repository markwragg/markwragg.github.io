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

This blog post is part of the [Azure Spring Clean 2025](https://learn.microsoft.com/en-us/azure/well-architected/) community event, promoting well managed Azure tenants. In last year's Azure Spring Clean, [Dan Rios blogged about using PSRule for Bicep code](https://rios.engineer/azure-spring-clean-azure-best-practice-for-bicep-with-psrule/). The focus of this blog post is on how you can use PSRule to validate resources deployed via [Terraform by HashiCorp](https://www.terraform.io/).

> Many DevOps tools were gifted to the Engineers, who above all else, desired automation. For within these tools was bound the strength and will to govern the Cloud.
> But they were all of them deceived, for another tool was made.
> In the land of Microsoft, in the fires of Mount Azure, the Architect [Bernie White](https://www.linkedin.com/in/bernie-white/) forged, in open source, a tool to validate all others.
> And into this tool he poured his creativity, his mastery and his determination to improve your infrastructure. 
> 
> One tool to [PSRule](https://microsoft.github.io/PSRule/v2/) them all.

I suspect it was something like that anyway.

[PSrule can be found in GitHub](https://github.com/microsoft/PSRule) and has been in development since December 2018. At time of writing the current stable version is v2.9.0 (v3 is in development). The [official website](https://microsoft.github.io/PSRule/v2/) has lots of guidance on getting started, and is officially described as:

> "A rules engine geared towards testing Infrastructure as Code (IaC). Rules you write or import perform static analysis on IaC artifacts such as: templates, manifests, pipelines, and workflows."

The idea is that you (or Microsoft, or the community) define rules for how your infrastructure should be configured and PSRule (executed as part of your development process) confirms those constraints are being followed. Infrastructure code is often complex, and can be developed by multiple individuals or teams over time. PSRule acts as a guard rail to ensure the infrastructure requirements of your organisation are being followed. 

Obviously developing these rules is itself a timely endeavour, but PSRule has done the heavy lifting for you by providing various pre-built rules based on best practice guidance such as the [Azure Well-architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/). PSRule is extensible, so you chose which existing rules you want to use, and customise them to meet your requirements, and/or develop your own rules to meet your specific requirements.

### Getting Started

[Installing PSRule for Azure](https://azure.github.io/PSRule.Rules.Azure/install/) requires no difficult journey to Mount Doom. Simply do the following:

- [Install PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell) (if you haven't already)
- Install PSRule (with the pre-built Azure rules) from the PowerShell Gallery:
  
```powershell
Install-Module -Name 'PSRule.Rules.Azure' -Repository PSGallery -Scope CurrentUser
```

