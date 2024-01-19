---
title: Unleashing Azure Cost Management with the AzCostTools PowerShell Module
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/Cloud-Computing-Cost-Management4.jpeg"
  teaser: "/content/images/2024/Cloud-Computing-Cost-Management4.jpeg"
date: '2024-01-16 14:00:00'
tags:
- powershell
- azure
---

Microsoft Azure is a powerful cloud computing platform that enables businesses to get started quickly, however as organizations embrace the cloud, managing costs becomes a critical aspect of optimizing resources and ensuring fiscal responsibility. I've been responsible for the costs of a number of Azure subscriptions for several years. While the built-in cost management tools are useful (and I strongly recommend you use them), I also turned to PowerShell so that I could extract consumption data and perform comparisons of how costs changed month to month.

I've recently developed that script into a public tool with additional functionality and it is now available on GitHub and the PowerShell Gallery within a module called [AzCostTools](https://github.com/markwragg/PowerShell-AzCostTools). See below for some examples of what it can do.

> If you're interested in some general advice regarding how to approach managing costs in Azure, see my [Azure Cost Management](https://mpfe.uk/blog/2023-03-31-azure-cost-management/) company blog post. 

## Source Code
 
If you'd like to review the source code, you can find it on GitHub:

- https://github.com/markwragg/PowerShell-AzCostTools/

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

Finally ensure you're logged in to Azure via the AZ PowerShell module, and that your current Azure context has access to the subscriptions you wish to query:

```powershell
Connect-AzAccount
Get-AzContext
```

> The module assumes you have access to return cost data for the subscriptions you specify. An error will be returned if you do not.
>
> Some subscriptions have costs managed elsewhere, for example if your Azure tenant or subscription is provided via a Cloud Solutions Provider (CSP) this module cannot currently access your costs.

There's just two public cmdlets in the module currently:

- `Get-SubscriptionCost`
- `Show-CostAnalysis`

## Get-SubscriptionCost

This is cmdlet retrieves cost data from Azure, by using the `Get-AzConsumptionUsageDetail` cmdlet. You can run it without any parameters:

```powershell
Get-SubscriptionCost
```

This will execute `Get-AzSubscription` to return all subscriptions you currently have access to in your current Azure context, and then it will query them one by one for cost data for the current billing month.

There's a default table view returned with the 


## Understanding the Need for AzCostTools

Managing costs in the cloud can be a complex task, with myriad services, configurations, and usage patterns contributing to the overall expenditure. **AzCostTools** emerges as a beacon in this landscape, offering a simplified yet robust approach to interpreting and visualizing your Azure cost data.

## Key Features of AzCostTools

1. **Seamless Integration:** AzCostTools seamlessly integrates with your Azure environment, harnessing the power of PowerShell to retrieve and process cost data effortlessly.

2. **Interactive Charts:** Say goodbye to mundane spreadsheets! AzCostTools empowers users with visually appealing and interactive charts, providing a bird's eye view of your Azure expenditure trends and patterns.

3. **Customizable Analysis:** Tailor your analysis to meet specific business needs. Whether you're focusing on resource-specific costs, departmental breakdowns, or historical comparisons, AzCostTools puts the control in your hands.

4. **Automated Reporting:** Save time and effort with automated reporting capabilities. Schedule regular reports to keep stakeholders informed and facilitate data-driven decision-making.

## Getting Started with AzCostTools

1. **Installation:** Getting started with AzCostTools is a breeze. Simply install the PowerShell module and connect it to your Azure subscription.

```powershell
Install-Module -Name AzCostTools -Force -AllowClobber
```

Data Retrieval:

```powershell
Connect-AzAccount
Import-Module AzCostTools
$costData = Get-AzCostData -StartDate '2024-01-01' -EndDate '2024-01-15'
```

Generate Charts:

```powershell
$costData | Show-AzCostChart -ChartType 'Line' -GroupBy 'Service'
```

## Explore Insights:

Dive into your cost data, explore insights, and make informed decisions about optimizing your Azure resources.

## Why AzCostTools?

* Empower Decision-Making: Gain a deeper understanding of your Azure costs to make informed decisions about resource allocation, scaling, and budgeting.

* Enhance Visibility: AzCostTools enhances visibility into your cloud expenditure, allowing you to identify cost spikes, trends, and potential areas for optimization.

* Simplify Audits: Facilitate financial audits and compliance by presenting clear and comprehensive reports on your Azure spending.

## Conclusion

AzCostTools is more than just a PowerShell module; it's a game-changer for organizations seeking to master Azure cost management. As the cloud landscape continues to evolve, staying in control of your costs becomes paramount. AzCostTools empowers you to do just that, providing a robust solution to transform your Azure cost data into actionable insights.

Embark on a journey of efficient cost management with AzCostTools. Install the module, explore the features, and unlock a new level of control over your Azure expenditure. The cloud is vast, but with AzCostTools, your financial landscape becomes clear and manageable.