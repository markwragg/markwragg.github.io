---
layout: post
title: Managing SSRS Subscriptions with PowerShell
image: "/content/images/2017/10/BusinessAutomation.jpg"
date: '2017-10-28 07:41:09'
tags:
- powershell
- automation
- ssrs
---

I was recently tasked with investigating how we could store the configuration of our SQL Server Reporting Services report subscriptions  in a source control and then automate the process of (later) configuring them in one or more new SSRS servers.

My first instinct was to look for a PowerShell module and I discovered a [Microsoft maintained module named ReportingServicesTools](https://github.com/Microsoft/ReportingServicesTools) in Github and published to the Powershell Gallery.

If you have Powershell 5 or newer you can install the module with:

    Install-Module ReportingServicesTools

For older versions of PowerShell the module has an `install.ps1` script that you can invoke as follows:

    Invoke-Expression (Invoke-WebRequest https://aka.ms/rstools)

I was pleased to discover that [Cláudio Silva](https://claudioessilva.eu/) had recently contributed two cmdlets to the module that looked to meet my need:

- `Get-RsSubscription` -- retrieves subscriptions from a specified report path or folder. 
- `Set-RsSubscription` -- creates subscriptions that have been retrieved and piped in via the aforementioned command, in a specified server and folder.

These cmdlets were written to allow someone to clone reports or copy them between systems by simply doing:

    Get-RsSubscription -Path '/old/report' | Set-RsSubscription -Path '/new/report'

However unfortunately they only work if the action is done in a single session, exporting the result of `Get-RsSubscription` to disk via `Export-CliXML` (per my need) and then reimporting it with `Import-CliXML` made the resultant object invalid for the `Set-RsSubscription` cmdlet due to the object being deserialized and the SOAP API requiring strongly typed inputs.

---

**TL;DR:** As a result, I have [updated the ReportingServicesTools module](https://github.com/Microsoft/ReportingServicesTools/pull/79) to add cmdlets which enable subscriptions to be stored on disk:

- `Export-RsSubscriptionXml` -- exports subscriptions to disk using Export-CliXML with `-Depth` set to 3.
- `Import-RsSubscriptionXml` -- imports subscriptions from one or more XML files on disk and recreates a number of the properties to make them valid object types for `Set-RsSubscription`. 

I also had a need to retrieve subscriptions from a SQL Server 2008 SSRS system (which uses an older version of the SOAP API) and as such amended the module to add some backwards compatibility support for this API via an `-ApiVersion` parameter.

As a result you can now save subscriptions to disk by doing:

    Get-RsSubscription -Path '/old/report' -ApiVersion 2005 | Export-RsSubscription SomeFile.xml

*-- `-ApiVersion` is not needed if you're exporting from a SQL Server 2008 R2 or newer system.*

And import them in later by doing:

    Import-RsSubscription SomeFile.xml | Set-RsSubscription -Path '/new/report'

I've also added the following two cmdlets to allow subscriptions to be created from scratch directly with PowerShell:

- `New-RsSubscription` -- creates subscriptions in a specified location using settings defined by the cmdlets various parameters.
- `New-RsScheduleXml` -- use this helper function to generate the XML that you need to pass to the `New-RsSubscription -Schedule` parameter if you want to create a scheduled/recurring subscription.

Read on below for more in depth information about how these work.

> Were it not for Cláudio's cmdlets as well as the work of the others in the module this would have been much more difficult to figure out. I'm also really grateful for the help and support he offered me via Twitter, reviewing my code (while on holiday no less!).
>
> It's also great that Microsoft are maintaining things like this as open source and are supportive in helping new contributors understand how to meet the standards for the project.

---

SSRS uses a SOAP based API for interacting with the system config. It also has a REST API, but that is for interacting with SSRS setup/config. 

These new cmdlets (where applicable) interact with the SOAP API.

## Export-RsSubscriptionXml
[[Source code]](https://github.com/Microsoft/ReportingServicesTools/blob/master/ReportingServicesTools/Functions/CatalogItems/Export-RsSubscriptionXml.ps1)

```
SYNTAX
    Export-RsSubscriptionXml [-Path] <String> -Subscription <Object> [-WhatIf] [-Confirm] [<CommonParameters>]
```
This cmdlet really only exists to compliment the more complex and necessary `Import-RsSubscriptionXml` cmdlet described below. This cmdlet is just a wrapper around `Export-CliXML` that ensures you also set `-Depth 3`, as the default depth of 2 doesn't result in fully exporting all properties of the subscription.

**Usage Examples**:

Export the current set of subscriptions contained in '/path/to/my/report' to an XML file named MySubscriptions.xml: 
```
Get-RsSubscription -path '/path/to/my/report' | Export-RsSubscriptionXml .\MySubscriptions.xml
```

## Import-RsSubscriptionXml

[[Source code]](https://github.com/Microsoft/ReportingServicesTools/blob/master/ReportingServicesTools/Functions/CatalogItems/Import-RsSubscriptionXml.ps1)

```
SYNTAX
    Import-RsSubscriptionXml [-Path] <String> [-ReportServerUri <String>] [-Credential <PSCredential>] [-Proxy <Object>] [<CommonParameters>]
```

This cmdlet uses `Import-CliXml` to read subscriptions from one or more XML files on disk as saved by the above cmdlet. At this point the subscription object has been recreated, but a number of sub-properties are rendered invalid as input to the Reporting Services SOAP API due to being deserialized. As such, this cmdlet then recreates each of those properties. It does this by first creating a web proxy connection to the SOAP web service which it then uses to access the class and methods of the API, as [documented here](https://docs.microsoft.com/en-us/sql/reporting-services/report-server-web-service/methods/report-server-web-service-methods).

The properties that get recreated are:

- `.DeliverySettings`
  - `.ParameterValues` : these get recreated as either a `.ParameterValue` or a `.ParameterFieldReference` object.
- `.DataRetrievalPlan` (for Data Driven subscriptions) and it's sub properties:
  - `.Query`
  - `.DataSet`
  - `.Reference`
- `.Values`

The script simply loops through the values for each of these properties that were loaded from the XML file and then uses `New-Object` to create the corresponding object type needed to populate these new properties with the existing values.

When it has finished, it outputs the subscriptions as valid PowerShell objects, which means they can then be consumed by `Set-RsSubscription` via the pipeline.

**Usage Example:**

Import the subscriptions contained in .\MySubscriptions.xml, recreate any SSRS specific properties  and pipe the results to Set-RsSubscription which will add them to the /Example/Report report:
```
Import-RsSubscriptionXml .\MySubscriptions.xml | Set-RsSubscription -Path /Example/Report
```
## New-RsSubscription

[[Source Code]](https://github.com/Microsoft/ReportingServicesTools/blob/master/ReportingServicesTools/Functions/CatalogItems/New-RsSubscription.ps1)

This cmdlet allows you to create new subscriptions from scratch by defining the settings via it's parameters. 

There are different parameter set requirements depending on whether the destination of the subscription is a file share or email (as defined by the `-Destination` parameter). The function does not currently support setting SharePoint as a destination.

```
SYNTAX
    New-RsSubscription [-ReportServerUri <String>] [-Credential <PSCredential>] [-Proxy <Object>] -Path <String> [-Description <String>] [-EventType <String>]
    -Destination <String> -Schedule <String> -DestinationPath <String> -Filename <String> -RenderFormat <String> [-WhatIf] [-Confirm] [<CommonParameters>]

    New-RsSubscription [-ReportServerUri <String>] [-Credential <PSCredential>] [-Proxy <Object>] -Path <String> [-Description <String>] [-EventType <String>]
    -Destination <String> -Schedule <String> -To <String> [-CC <String>] [-BCC <String>] [-ExcludeReport] -Subject <String> -RenderFormat <String> [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

Similar to Import-RsSubscriptionXml, this is using the proxy to interact with the necessary class in order to build the appropriate object that the SOAP API requires for the [CreateSubscription](https://msdn.microsoft.com/library/reportservice2010.reportingservice2010.createsubscription(v=SQL.130).aspx) method.

**Usage Example:**

Create a subscription for the report/s in the specified `-Path` that emails them fortnightly on Saturday to the specified address in PDF format:

    New-RsSubscription -ReportServerUri http://yourserver/Reportserver -path /some/path -Description 'Report A Fortnightly by email' -Destination 'Email' -Schedule (New-RsScheduleXML -Weekly -Interval 2 -DaysOfWeek Saturday) -Subject 'Fortnightly Report A' -To 'test@someaddress.com' -RenderFormat 'PDF'


## New-RsScheduleXml

[[Source Code]](https://github.com/Microsoft/ReportingServicesTools/blob/master/ReportingServicesTools/Functions/CatalogItems/New-RsScheduleXML.ps1)

This cmdlet generates the XML that the `-Schedule` property requires in the `New-RsSubscription` cmdlet. The `CreateSubscription` method of the SOAP API requires this particular input be in XML and the XML needs to be structured in a specific way.

> This cmdlet is designed to work in a similar way to `New-ScheduledTaskTrigger` which is a helper cmdlet you use with `New-ScheduledTask` to create a valid input for it's `-Trigger` parameter.

You can generate a number of different schedule types (these same options are available via the UI when you create a subscription in SSRS): 

- Once: Runs once at a specified date/time.
- Minute: repeats every X minutes.
- Daily: repeats every X days.
- Weekly: repeats every X weeks and you an optionaly specity DaysOfWeek to make it repeat only on specified named days of the week.
- Monthly: repeats every month on the specified days (which is a string where you can provide specific dates or a range, e.g 1,3,5,10-15). You can also optionally specify the months this is valid for (e.g January,June,October) rather than every month.
- MonthlyDayOfWeek: repeats every month on a specific week of the month (e.g First, Second, Third, Last). You can also optionally specify the Days of the Week.

For each of these schedules you can also specify a Start and End date/time period during which the schedule is valid.

Here's the full Syntax from the `get-help`:

```
SYNTAX
    New-RsScheduleXml [-Once] [-Start <DateTime>] [-End <DateTime>] [-WhatIf] [-Confirm] [<CommonParameters>]

    New-RsScheduleXml [-Minute] [[-Interval] <Int32>] [-Start <DateTime>] [-End <DateTime>] [-WhatIf] [-Confirm] [<CommonParameters>]

    New-RsScheduleXml [-Daily] [[-Interval] <Int32>] [-Start <DateTime>] [-End <DateTime>] [-WhatIf] [-Confirm] [<CommonParameters>]

    New-RsScheduleXml [-Weekly] [[-Interval] <Int32>] [-DaysOfWeek <String[]>] [-Start <DateTime>] [-End <DateTime>] [-WhatIf] [-Confirm] [<CommonParameters>]

    New-RsScheduleXml [-Monthly] [-Months <String[]>] -DaysOfMonth <String> [-Start <DateTime>] [-End <DateTime>] [-WhatIf] [-Confirm] [<CommonParameters>]

    New-RsScheduleXml [-MonthlyDayOfWeek] [-DaysOfWeek <String[]>] [-Months <String[]>] -WeekOfMonth <String> [-Start <DateTime>] [-End <DateTime>] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**Usage Examples:**

Create an XML string to schedule a subscription to run once at the date/time specified:
```
New-RsScheduleXml -Once -Start '01/02/2017 12:34'
```
Create an XML string to schedule a subscription to run every 90 minutes starting at 9pm:
```
New-RsScheduleXml -Minute -Interval 90 -Start '21:00'
```
Create an XML string to schedule a subscription to run every 5 days starting at the current date/time:
```
New-RsScheduleXml -Daily -Interval 5
```
Create an XML string to schedule a subscription to run every 5 days starting at the current date/time:
```
New-RsScheduleXml -Weekly -Interval 2 -DaysOfWeek Monday,Tuesday,Wednesday,Thursday,Friday
```
---
## Limitations

There are a few limitations to the cmdlets at the moment to be aware of:

- `New-RsSubscription` doesn't currently support creating subscriptions within reports that have one or more parameters (defined in the report).
- `New-RsSubscription` doesn't currently support the creation of data driven subscriptions.
- `Set-RsSubscription` does not support creating subscriptions using the older API version (you can just retrieve them with the `Get-` cmdlet).

If you find any other issues/limitations to the cmdlets please raise them as [issues in GitHub](https://github.com/Microsoft/ReportingServicesTools/issues).

## Useful Links

If you are looking to make modifications to the [ReportingServicesTools](https://github.com/Microsoft/ReportingServicesTools) module or are looking to develop solutions around the SSRS API, these links may be helpful:

- http://www.tek-tips.com/viewthread.cfm?qid=1780331
- https://sqlblogcasts.com/blogs/sqlandthelike/archive/2013/02/12/deploying-ssrs-artefacts-using-powershell-simply.aspx
- http://www.techtree.co.uk/windows-server/windows-powershell/powershell-ssrs-function-export-ssrsreports-to-rdl-files/
- https://msdn.microsoft.com/en-us/library/reportservice2010.reportingservice2010.aspx
- http://odetocode.com/Articles/114.aspx
- http://www.sqlservercurry.com/2009/07/programmatically-create-data-driven.html
- http://jaliyaudagedara.blogspot.co.uk/2012/10/creating-data-driven-subscription.html
- https://www.codeproject.com/Questions/1034771/SSRS-Programmatically-Create-Subscriptions
- http://www.sqlservercentral.com/blogs/cl%C3%A1udio-silva/2017/07/27/does-that-copy-subscriptions-too-now-it-does-new-powershell-ssrs-commands/
- https://www.mssqltips.com/sqlservertip/4738/powershell-commands-for-sql-server-reporting-services/
- https://docs.microsoft.com/en-us/sql/reporting-services/report-server-web-service/accessing-the-soap-api
- https://www.codeproject.com/Articles/36009/Programmatically-Playing-With-SSRS-Subscriptions