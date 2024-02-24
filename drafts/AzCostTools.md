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

**Should cloud computing be illegal?** Probably not, _however_ it is incredibly easy to get started, difficult to stop, and before you know it you could be selling your grandmother just to afford one more month of compute. Hopefully your circumstances never get that dire, but I've certainly seen plenty of companies that have entrenched themselves into the highly addictive world of automated, scalable infrastructure, but then struggle to understand the sometimes astronomical monthly bill.

I found myself in this situation some time ago (not selling my grandmother.. but trying to understand high bills). After some wrangling, I managed to cut a client's cloud bill by 60%, saving approximately £500,000 over 2 years, and that was despite [Microsoft increasing their prices by 11% last April](https://news.microsoft.com/europe/2023/01/05/consistent-global-pricing-for-the-microsoft-cloud/). 

Rather than a 12-step program, I think cloud addiction can be treated in just 6:

1. Understand your costs (use the built-in tools)
2. Take ownership of the costs and perform regular reviews
3. Configure budgets and billing alerts
4. Automate the creation and destruction of your resources (and get good at it)
5. Ensure the purpose/ownership of all resources is understood
6. Purchase reservations

I've [blogged about these topics before](https://mpfe.uk/blog/2023-03-31-azure-cost-management/) on my company website. However in this blog post I will be focussing on #2: take ownership of the costs and perform regular reviews, by way of introducing a PowerShell module I've recently published to help do just that.

You may want to review your costs more frequently than monthly, but unless you have a very static environment, I think its a good minimum guideline as its how Azure usage is billed. Per step 3, it's important to also have budgets and billing alerts configured (and with thresholds that are close to your typical costs) so that if your usage spikes unexpectedly during the month you are made aware and can intervene if appropriate.

My AzCostTools module helps with monthly reviews by:

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

You'll need to also ensure you have the [AZ modules](https://learn.microsoft.com/en-us/powershell/azure/install-azure-powershell?view=azps-11.3.0) (so that you have `Get-AzConsumptionUsage` and some other cmdlets that the module uses). If you want to generate charts, you also need to install a module called [PSparklines](https://github.com/endowdly/PSparklines). AzCostTools will work without it, but the charts are both vaguely informative and fun.

To install these prerequisites, execute:

```powershell
Install-Module -Name Az #Assuming you don't already have it installed
Install-Module -Name PSparklines
```

Finally you of course need to make sure you've authenticated to Azure via the Az module, and for the tenant/s that you want to query costs. To login to Azure execute:

```powershell
Connect-AzAccount
```

You are now ready to start querying your costs. So brace yourselves, this might hurt.

### Retrieving costs

Ensure you have logged in to AZ PowerShell via `Login-AzAccount` and to the tenant that has the Subscription/s you wish to query.

> Depending on the number of subscriptions and/or previous months of data you wish to query the `Get-SubscriptionCost` cmdlet can take a few minutes to run.
> I recommend you return the results to a variable. If you want to see the output while also saving to a variable, use the `-OutVariable` parameter.
> E.g: `Get-SubscriptionCost -OutVariable Cost` will return the results to `$Cost` while also showing them on screen.

To return cost data for the current billing month for all Subscriptions in your current Azure Context, execute:

```powershell
Get-SubscriptionCost
```

> Errors may be returned for any subscriptions where the cost data is inaccessible, e.g you are not authorised to access costs or the subscription is of a type where costs are managed externally (such as a CSP).

There is a default table view. Pipe the result to `Format-List` to see all of the properties that are returned.

![Get-SubscriptionCost returns current costs for all subscriptions in the current context](/content/images/2024/Get-SubscriptionCost.png)

To return cost data for the current billing month for a specified subscription, and compare those costs to the previous billing month, execute:

```powershell
Get-SubscriptionCost -SubscriptionName 'AdventureWorks Cycles' -ComparePrevious -SparkLineSize 3
```
> In the above example we also increased the size of the charts by specifying `-SparkLineSize`.

![Get-SubscriptionCost returns costs for a specified subscription and compares them to the previous month with sparkline charts that are 3 rows in height](/content/images/2024/Get-SubscriptionCost-ComparePrev.png)

To return a number of previous months, you can use the `-PreviousMonths` parameter. For example:

```powershell
Get-SubscriptionCost -PreviousMonths 5 -ComparePrevious
```

> In the above example we've also used `-ComparePrevious` so that for each month calculations are made comparing it to the previous month. This is optional.

![Get-SubscriptionCost returns current costs for all subscriptions in the current context and the previous 5 months of costs](/content/images/2024/Cost-MultipleSubscription-PrevMonths-ComparePrev.png)

When using `-ComparePrevious` you can also specify `-ComparePreviousOffset`. This will compare each month of cost data returned to X month/s prior as specified.
For example, if you wanted to compare costs for the last 6 months against the same 6 months from the year prior, you could execute:

```powershell
Get-SubscriptionCost -PreviousMonths 6 -ComparePrevious -ComparePreviousOffset 12
```
![Get-SubscriptionCost returns current costs for all subscriptions in the current context and the previous 6 months of cost, comparing each to the equivalent month 12 months prior](/content/images/2024/Cost-MultipleSubscription-PrevMonths-ComparePrev-Offset12.png)

Other parameters available for `Get-SubscriptionCost` include:

* `-BillingMonth` — Use to specify a specific month to retrieve costs (or as a starting point from when also retrieving previous months costs).
* `-Raw` — Adds properties to the resultant object that include the raw cost data returned by `Get-AzConsumptionUsageDetail` in case you want to do further direct analysis/manipulation.

### Cost Analysis

Having retrieved a set of cost data for one or more subscriptions, you can pipe that data to `Show-CostAnalysis` to generate charts and tables analysing the costs:

```powershell
$Cost | Show-CostAnalysis
```

If `PSparklines` is installed, a daily cost chart will be generated. If the subscription has a budget this will show red for days over budget, and green for under (based on a daily budget calculation).
If there is no budget for the subscription the chart will be white.

A chart and table is also generated of the top 15 service costs, with each service name mapped to an individual colour.

If more than one subscription is in the cost data, the cmdlet will end with a total of cost for all subscriptions and a chart showing most to least expensive.

![Show-CostAnalysis generates charts and tables for a set of returned cost data](/content/images/2024/Show-CostAnalysis.gif)

With `Show-CostAnalysis` you can also customise the size of the charts returned by specifying `-SparkLineSize`. The default is 3.
You can also specify `-ConvertToCurrency` with a 3 letter currency code if you'd like the cost values returned to be converted to a different currency. 
Sometimes Azure costs are billed in a currency that is not your own and it may be more informative to view them in your local currency. For example:

```powershell
$Cost | Show-CostAnalysis -ConvertToCurrency GBP
```

> Note that this uses a free/open API for currency conversion that only refreshes the exchange rates once a day.

If you used `-ComparePrevious` when executing `Get-SubscriptionCost` you can also specify `-ComparePrevious` for `Show-CostAnalysis` to generate further tables and charts for the previous cost data. This might be most useful when using `-ComparePreviousOffset` so that you can see the charts side by side of the current and previous costs. For example:

```powershell
$Cost | Show-CostAnalysis -ComparePrevious
```
![Show-CostAnalysis generates charts and tables for a set of returned cost data and shows charts for the previous cost data](/content/images/2024/Show-CostAnalysis-ComparePrev.png)
