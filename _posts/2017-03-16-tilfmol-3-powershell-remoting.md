---
title: 'TILFMOL #3 - PowerShell Remoting'
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2017/02/global-network-1.jpg"
  teaser: "/content/images/2017/02/global-network-1.jpg"
date: '2017-03-16 21:29:50'
tags:
- powershell
---
This is part three of a short series of posts about things I discovered from reading [Learn PowerShell in a Month of Lunches](https://www.manning.com/books/learn-windows-powershell-in-a-month-of-lunches-third-edition) (recently released in 3rd edition).

This post focuses on things I learnt about PowerShell remoting, including:

- Executing remote commands on one or many machines
- Deserialised objects are the result of commands
- Creating Endpoints

# Quick Intro to Remoting

PowerShell Remoting provides a standard way of executing cmdlets remotely and retrieving the results. It means [that the communication ports required are standardised](https://blogs.msdn.microsoft.com/wmi/2009/07/22/new-default-ports-for-ws-management-and-powershell-remoting/) and cmdlet developers don't need to code their own remote execution in to their command (that's not to say some developers still don't). Some built-in cmdlets in PowerShell have a `-computername` parameter and this is often because they date from the pre-Remoting era, but PS Remoting now makes that generally unnecessary.

*-- If you want to play around with PS Remoting it may first need to be enabled. This is usually a simple case of entering `enable-psremoting` in a PowerShell window (with appropriate privileges).*

*There can be some complexities in getting it working if your source and destination machines aren't on the same domain or are separated by a complex network.*

![Enabling PowerShell Remoting](/content/images/2017/03/Enable-PSRemoting.png)

## One-to-one remoting

Once you have PS Remoting enabled, the simplest way to use it is use the `Enter-PSSession` and `Exit-PSSession` cmdlets, which allow you to open and connect to a PowerShell session on a remote computer (similar to SSH or a command-line only RDP). Your prompt changes to include the remote server name so you can tell where you are.

## One-to-many remoting

The true power of remoting is the ability to run a command and have it execute simultaneously on multiple machines (by default up to 32 at once). You do this with the `Invoke-Command` cmdlet which has a `-ComputerName` parameter that accepts one or more names and a `-ScriptBlock` parameter that is the block of code that you want to execute (or with an alternative parameter you can use a script file). 

*-- You can of course also use `Invoke-Command` to run commands on a single machine, without the need to open an interactive session per `Enter-PSSession`. You can also use Invoke-Command on a local computer to evaluate or run a string in a script block as a command.*

Here's an example usage of `Invoke-Command` which return the latest 10 System log entries from three servers:
```
Invoke-Command -ComputerName Server1,Server2,Server3 -ScriptBlock { 
    Get-EventLog System -Newest 10
}
```

The [official documentation for Invoke-Command](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.core/invoke-command) has lots of great explanations and examples of the different uses.

# Deserialised objects

The results returned from remote commands are deserialised which just means they are static and have no methods to enable interactions (as they do when retrieved locally). This isn't particularly surprising, but it was interesting to have called out in MOL.

You can see this is the case by using the trusty `| get-member` command on your results, by which you will observe the lack of methods (except `ToString()`) but also that the typename at the top of the object has changed to be `Deserialized.whatever.object`. 

For example:

```
PS C:\> $s = New-PSSession localhost
PS C:\> Invoke-Command $s { Get-Process } | Get-Member


   TypeName: Deserialized.System.Diagnostics.Process
```

The takeaway from this is that if you need/want to execute the method of an object (and assumedly you want to do so on the machine it originated from) you generally want to make sure you do so remotely (as part of the `invoke-command` scriptblock), before the result is returned.

There's a great blog post about [how objects are sent to and from remote sessions](https://blogs.msdn.microsoft.com/powershell/2010/01/07/how-objects-are-sent-to-and-from-remote-sessions/) by the PowerShell team that covers this in more detail.

# Creating Endpoints

> "A computer can contain multiple endpoints, which PowerShell refers to as session configurations. For example, enabling remoting on a 64-bit machine enables an endpoint for 32-bit PowerShell as well as for 64-bit PowerShell, with 64-bit being the default." 
>
> -- Learn PowerShell in a Month of Lunches

You can see what endpoints currently exist on your machine by running `Get-PSSessionConfiguration` from an Administrator-privileged PowerShell window: 

![](/content/images/2017/03/PowerShell-Endpoints.png)

You can also create your own custom endpoints. This is a two stage process:

1. Run `New-PSSessionConfigurationfile` to create a config file with a .PSSC extension that defines the characteristics of the endpoint (e.g which commands and capabilities it includes).
2. Use `Register-PSSessionConfiguration` to load the .PSSC file and create the endpoint with the WinRM service. During registration you can set other parameters such as who may connect to the endpoint.

MOL points out that is likely most useful for delegated administration, where you want to give a set of users (such as your Tier 1 / Service Desk) a subset of commands to run.

In an increasingly security sensitive world, this is a growing topic and Microsoft have more recently been building on this with a project titled [Just Enough Administration](https://msdn.microsoft.com/en-us/powershell/jea/overview) which is also an [open source project in GitHub](https://github.com/PowerShell/JEA).

---
This has obviously been a very high-level overview of some of the remoting features of PowerShell and I hope it inspires you to read further. 

In my next and final chapter for this series I to look at [PowerShell Jobs](http://wragg.io/tilfmol-4-powershell-jobs/), the background task engine of PowerShell.

Here are some links in case you missed my previous posts in this series on the [PowerShell Pipeline](http://wragg.io/tilfmol1-the-powershell-pipeline/) and the [PowerShell Help System](http://wragg.io/tilfmol-2-powershell-help/).
