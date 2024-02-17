---
title: Monitor and manage your Azure costs with a little help from PowerShell

header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/Cloud-Computing-Cost-Management4.jpeg"
  teaser: "/content/images/2024/Cloud-Computing-Cost-Management4.jpeg"
date: '2024-02-16 14:00:00'
tags:
- powershell
- azure
- azurespringclean
- module
- costmanagement
---

**Cloud computing should be illegal.** It's incredibly easy to get started, but before you know it you're selling your grandmother just so you can afford another month of compute. Hopefully your circumstances aren't that extreme, but I've certainly seen plenty of companies that have entrenched themselves into the highly addictive world of automated, scalable infrastructure, but struggle to understand the sometimes astronomical monthly bill.

I found myself in this situation some time ago (not selling my grandmother.. but trying to understand high bills). After some wrangling, I managed to cut a client's cloud bill by 60%, saving approximately £500,000 over 2 years, despite [Microsoft increasing their prices by 11% in April](https://news.microsoft.com/europe/2023/01/05/consistent-global-pricing-for-the-microsoft-cloud/). 

Rather than a 12-step program, I think cloud addiction can be treated in just 6:

1. Understand your costs (use the built-in tools)
2. Take ownership of the costs and perform regular reviews
3. Configure budgets and billing alerts
4. Automate the creation and destruction of your resources (and get good at it)
5. Ensure the purpose/ownership of all resources is understood
6. Purchase reservations

I've [blogged about these topics before](https://mpfe.uk/blog/2023-03-31-azure-cost-management/) on my company website. However in this blog post I will be focussing on #2: take ownership of the costs and perform regular reviews, by way of introducing a PowerShell module I've recently published to help do just that.

You may want to review your costs more frequently than monthly, but unless you have a very static environment, I think its a good minimum guideline as its how Azure usage is billed. Per step 3, it's important to also have budgets and billing alerts configured (and with thresholds that are close to your typical costs) so that if your usage spikes unexpectedly during the month you are made aware and can intervene if appropriate.

My [AzCostTools]() module helps with monthly reviews by:

- Extracting your cost data via the `Get-AzConsumptionUsage` cmdlet for one or more months
- Comparing those months to the previous ones (or to a previous month of your choice by configuring an offset).
- Calculating the cost difference and change percentage
- Charting the daily cost, so you have a basic view of how costs fluctuate
- Retrieves any budgets you have configured and indicates where you've been within/over budget
- Identifying the most expensive services, by type

Per step 1, this tool is not intended to replace the built-in cost management tools that are in the Azure Portal. I strongly advocate their use, and there's a lot of built-in views that are easy to use. My personal favourite is the daily cost view, granulated by Resource Group so that I can drill into where the most expensive resources are. But if you're responsible for more than one tenant and/or multiple subscriptions, you may find AzCostTools works well to automate the reporting of those costs. Another weakness of the built-in Cost Management interface is it only seems to be able to show you the last 12 months of costs. If you wanted to compare your costs for the last few months to their same month a year ago (which might be a reasonable thing to do if your costs fluctuate seasonally), its not very helpful. However the `Get-AzConsumptionUsage` cmdlet does return data from more than 12 months ago, and so AzCostTools can do this comparison for you.

That's probably enough selling (it's free btw..), here's how to use it:

The module is in GitHub, so you can have a look at the source code here:

- [https://github.com/markwragg/PowerShell-AzCostTools/](https://github.com/markwragg/PowerShell-AzCostTools/)

You can install the module from the [PowerShell Gallery](https://www.powershellgallery.com/packages/AzCostTools/) so to get started, open a PowerShell window and execute:

```powershell
Install-Module -Name AzCostTools
```

You'll need to also ensure you have the AZ module (so that you have `Get-AzConsumptionUsage` and some other cmdlets that the module uses). If you want to generate charts, you also need to install a module called `PSparklines`. AzCostTools will work without it, but charts are fun.

To install these prerequisites, execute:

```powershell
Install-Module -Name AZ #Assuming you don't already have it installed
Install-Module -Name PSparklines
```