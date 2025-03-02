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

This blog post is part of the [Azure Spring Clean 2025](https://learn.microsoft.com/en-us/azure/well-architected/) community event, promoting well managed Azure tenants. In last year's Azure Spring Clean, [Dan Rios blogged about using PSRule for Bicep code](https://rios.engineer/azure-spring-clean-azure-best-practice-for-bicep-with-psrule/). The focus of this blog post is on how you can use PSRule to validate Azure resources deployed via [Terraform by HashiCorp](https://www.terraform.io/).

> Many DevOps tools were gifted to the Engineers, who above all else, desired automation. For within these tools was bound the strength and will to govern the Cloud.
> But they were all of them deceived, for another tool was made.
> In the land of Microsoft, in the fires of Mount Azure, the Architect [Bernie White](https://www.linkedin.com/in/bernie-white/) forged, in open source, a tool to validate all others.
> And into this tool he poured his creativity, his mastery and his determination to improve your infrastructure.
>
> One tool to [PSRule](https://microsoft.github.io/PSRule/v2/) them all.

I suspect it was something like that anyway.

[PSrule can be found in GitHub](https://github.com/microsoft/PSRule) and has been in development since December 2018. The current stable version is v2.9.0 (v3 is in development). The [official website](https://microsoft.github.io/PSRule/v2/) has lots of guidance on getting started, and is officially described as:

> "A rules engine geared towards testing Infrastructure as Code (IaC). Rules you write or import perform static analysis on IaC artifacts such as: templates, manifests, pipelines, and workflows."

The idea is that you (or Microsoft, or the community) define rules for how your infrastructure should be configured and PSRule (executed as part of your development process) confirms those constraints are being followed. Infrastructure code is often complex, and can be developed by multiple individuals or teams over time. PSRule can act as a guard rail to ensure the infrastructure requirements of your organisation are being followed, or used to evaluated the quality of existing infrastructure.

Obviously developing these rules is itself a timely endeavour, but PSRule has done the heavy lifting for you by providing various pre-built rules based on best practice guidance such as the [Azure Well-architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/). PSRule is extensible, so you choose which existing rules you want to use, and customise them (and/or develop your own rules) to meet your requirements.

### Begin the journey

[Installing PSRule for Azure](https://azure.github.io/PSRule.Rules.Azure/install/) requires no difficult journey to Mount Doom. There is a [dedicated site for this module](https://azure.github.io/PSRule.Rules.Azure/), with its own [Getting started](https://azure.github.io/PSRule.Rules.Azure/about/) page. You can install it directly, as follows:

- [Install PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell) (if you haven't already)
- Install PSRule (with the pre-built Azure rules) from the PowerShell Gallery:

```powershell
Install-Module -Name 'PSRule.Rules.Azure' -Repository PSGallery -Scope CurrentUser
```

PSRule for Azure is officially described as:

> "A pre-built set of tests and documentation to help you configure Azure solutions. These tests allow you to check your Infrastructure as Code (IaC) before or after deployment to Azure. PSRule for Azure includes unit tests that check how Azure resources defined in **ARM templates or Bicep code** are configured."

As it turns out, one does not simply test Terraform with PSRule.

![One does not simply test Terraform meme](/content/images/2025/one-does-not-simply-test-terraform.jpg){: .align-center}

But it's not _that_ complicated. At the moment you can't perform static analysis of your Terraform files (or Terraform plan output), but you can [indicate your support for the feature here](https://github.com/Azure/PSRule.Rules.Azure/issues/1193). For now, you need to deploy resources to Azure, run a command to export the configuration of those resources to a JSON file, which PSRule can then analyse. While this doesn't make PSRule as useful as it is for analysing Bicep (for which there is also a [VSCode extension](https://marketplace.visualstudio.com/items?itemName=bewhite.psrule-vscode) so you can get feedback while developing), if you deploy your infrastructure through a series of environments for testing (or even if you don't) I can see PSRule still forming a valuable part of that pipeline.

To summarise, here's the steps to take to analyse Terraform resources:

1. Deploy your Terraform code to Azure (ideally to a non-production environment).
2. Ensure you have authenticated to Azure via `Connect-AzAccount` and have the relevant subscription selected.
3. Export your Azure resources to a specified directory (and ensure the directory exists):

```powerShell
New-Item -Path out -ItemType Directory
Export-AzRuleData -OutputPath "$pwd/out"
```

By default this will export all of the resources for the currently selected subscription. You can use `-Subscription` to specify one or more subscriptions, or `-ResourceGroupName` to specify one or more resource groups. You can also use `-Tag` to export resources with one or more specified tags.

If you want to go big (or go home to the shire), you can use `-All` to export resources from all the subscriptions your current context can access.

The output file/s are named with the guid of the subscription. This is true even if you filter to specific resources by Resource Group or Tag, and any existing file with the same name will be overwritten.

### Face your challenges

> "It is the small things, everyday deeds of ordinary folk that keep the darkness at bay."

It's now time to analyse the output. You do this with `Invoke-PSRule` which you need to point at your exports and the `PSRule.Rules.Azure` module:

```powershell
Invoke-PsRule -InputPath "$pwd/out" -Module 'PSRule.Rules.Azure'
```

Depending on your resources, you'll probably get quite a lot of output because by default the tool returns all results (pass, fail and error). If you want to just see failed results, execute:

```powershell
Invoke-PSRule -InputPath "$pwd/out" -Module 'PSRule.Rules.Azure' -Outcome Fail
```

### Journey onward

