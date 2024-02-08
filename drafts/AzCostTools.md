---
title: Monitor and manage your Azure costs with a little help from PowerShell

header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/Cloud-Computing-Cost-Management4.jpeg"
  teaser: "/content/images/2024/Cloud-Computing-Cost-Management4.jpeg"
date: '2024-01-16 14:00:00'
tags:
- powershell
- azure
---

Microsoft Azure is a powerful cloud computing platform that enables businesses to get started quickly, however as organizations embrace the cloud, managing costs becomes a critical aspect of optimizing resources and ensuring fiscal responsibility. Several years ago I assumed responsibility for the costs of a number of Azure subscriptions. While I found the built-in cost management tools extremely useful, I also turned to PowerShell so that I could extract consumption data and perform comparisons of how costs changed month to month. I've recently developed that script into a public tool with additional functionality and it is now available on GitHub and the PowerShell Gallery as a module called [AzCostTools](https://github.com/markwragg/PowerShell-AzCostTools). See below for some examples of what it can do.

> If you're interested in some general advice regarding how to approach managing costs in Azure, see my [Azure Cost Management](https://mpfe.uk/blog/2023-03-31-azure-cost-management/) company blog post. 

> If you'd like to look at the the code for this module, you can find it on GitHub here: https://github.com/markwragg/PowerShell-AzCostTools/

## Installation

To get started, install the AzCostTools module from the PowerShell Gallery:

```powershell
Install-Module AzCostTools
```

You will also need to ensure you have the [AZ PowerShell module](https://learn.microsoft.com/en-us/powershell/azure/new-azureps-module-az) installed (which can also be installed via the PS Gallery):

```powershell
Install-Module AZ
```

 Next, I recommend that you install the PSparklines module. While AzCostTools will work without it, PSparklines allows the generation of some simplistic but informative visualisations by the way of SparkLine charts:

```powershell
Install-Module PSparklines
```

Finally ensure you're logged in to Azure via the AZ PowerShell module, and that your current Azure context has access to the subscription/s you wish to query:

```powershell
Connect-AzAccount
Get-AzContext
```

> The module assumes you have access to return cost data for the subscriptions you specify. An error will be returned if you do not.
>
> Some subscriptions have costs managed elsewhere, for example if your Azure tenant or subscription is provided via a Cloud Solutions Provider (CSP) this module cannot currently access your costs.

There's three cmdlets in the module currently:

- `Get-SubscriptionCost`
- `Get-StorageCost`
- `Show-CostAnalysis`

## Get-SubscriptionCost

This is cmdlet retrieves cost data from Azure, by using the `Get-AzConsumptionUsageDetail` cmdlet. You can run it without any parameters:

```powershell
Get-SubscriptionCost
```

This will execute `Get-AzSubscription` to return all subscriptions you currently have access to in your current Azure context, and then it will query them one by one for cost data for the current billing month.

There's a default table view returned with the 