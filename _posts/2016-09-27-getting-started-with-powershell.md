---
title: Getting started with Powershell
header:
  image: "/content/images/2016/09/getting-started-crop.jpg"
date: '2016-09-27 09:21:39'
tags:
- powershell
---
This post is a list of resources and tips to help anyone new to Windows Powershell in getting started with the language. If Powershell is completely new to you, I recommend you review all of the listed resources. If you find other great resources along the way, please feel free to comment below and i'll contribute them to this list.

# Getting Started

[Getting Started with PowerShell 3.0 Jump Start](https://mva.microsoft.com/en-us/training-courses/getting-started-with-powershell-3-0-jump-start-8276?l=r54IrOWy_2304984382)
(video series)

> This Jump Start Microsoft PowerShell course is designed to teach busy IT professionals, admins, and help desk persons about how to use PowerShell to improve management capabilities, automate redundant tasks, and manage the environment in scale. Through this PowerShell tutorial, learn how PowerShell works – and how to make PowerShell work for you – from experts Jeffrey Snover, the inventor of PowerShell, and Jason Helmick, Senior Technologist at Concentrated Technology.

Although this was released around Powershell 3 (and we're currently on version 5) this is still a very relevant introduction to Powershell, as each new version of Powershell builds on the last.

[My 12 PowerShell Best Practices](http://windowsitpro.com/blog/my-12-powershell-best-practices) (Blog Series)

> A 12-part series of posts entitled "What To Do / Not To Do in PowerShell." These summarize my 12 major best practices for the shell, many of which are also enthusiastically endorsed by the broader PowerShell community.

This promotes some essential best practices that will make your scripts more robust and easier for others to read/understand, so it's a good idea to develop these habits early.

[Powershell Basic Cheat Sheet](http://ramblingcookiemonster.github.io/images/Cheat-Sheets/powershell-basic-cheat-sheet2.pdf) (PDF)

> PowerShell is a task based command line shell and scripting language. To run it, click Start, type PowerShell, run PowerShell ISE or PowerShell as Administrator. Commands are written in verb-noun form, and named parameters start with a dash.

This is worth printing and keeping nearby when coding as a handy reference guide to the basic constructs and conventions.

[How to Install Windows 2016 TP5](How to Install Windows 2016 TP5) (Blog Post)

> This blog post (by me :) ) takes you through how to road test Windows Server 2016 Technical Preview 5 using VMWare Player as a desktop hypervisor. Note that you'll need to do this on a machine with a fair amount of spare RAM.
While this isn't strictly a Powershell resource, having a server VM to play around with is useful when learning Powershell.

This earlier blog post details how to install the trial version of the latest Windows Operating system where you can play around with the latest version of Powershell on the latest OS. You might want to spin up multiple machines if you want to experiment with using Powershell to communicate between servers.

[Powerscripting Podcast](https://powershell.org/podcast/) (Podcast)

> Shortly after PowerShell's introduction to the world, Jon Walz and Hal Rottenberg launched the PowerScripting Podcast, a weekly show featuring expert guest speakers, fantastic behind-the-scenes information, members of the PowerShell community, and more.

The more recent shows tend to be interviews on developments in Powershell and the community, but go back to episode 1 and the first 20 or so episodes do a good job of explaining and exploring the basics and this is a good way to use dead time such as a commute to immerse yourself in Powershell.

[Learn Powershell in a Month of Lunches](https://www.amazon.co.uk/Learn-Windows-PowerShell-Month-Lunches/dp/1617290211) (Book / eBook)

> *Learn Windows PowerShell in a Month of Lunches* is an innovative tutorial designed for busy administrators. Author Don Jones has taught thousands of administrators to use PowerShell, and now he brings his years of training techniques to a concise, easy-to-follow book. Just set aside one hour a day—lunchtime would be perfect—for an entire month, and readers will be automating administrative tasks faster than they ever thought possible.

I have not personally read it, but this book is recommended almost everywhere as a great way to learn Powershell.

# General Tips
- Google is your friend, but beware that as parameters start with a hyphen you might need to enclose some of your search with speechmarks as using a hyphen in a Google search means to exclude that term by default. E.g: instead of get-aduser -properties do get-aduser "-properties"
- StackExchange is your other friend and someone has probably already asked your question: http://stackoverflow.com/questions/tagged/powershell
- The console and ISE both support tab completion, so you can type part of a cmdlet name then tab through the options. Also the ISE has intellisense, it will show tooltips to help you complete commands and it will red underline coding errors.
- Get-Help is sometimes helpful (although Technet or ss64 are likely easier to read), e.g get-help get-service. Also try using it with any of these parameters: -examples -detailed or -full.
- Powershell is all about the pipeline. Cmdlets return collections of results known as "objects" that can be passed forward down the pipeline in to other cmdlets to manipulate the result set. In particular, get to grips with where-object, select-object and foreach-object (and bear in mind these are sometimes used via their "alias" which are ?, select and % respectively) 
- Get-Member is incredibly helpful, as it shows you all the properties and methods for an object (and can help you to understand what type of object you are handling). To use get-member just pipe an object to it. For example:
```
PS C:\Users\mwragg> get-service | get-member


   TypeName: System.ServiceProcess.ServiceController

Name                      MemberType    Definition
----                      ----------    ----------Name                      AliasProperty Name = ServiceName
RequiredServices          AliasProperty RequiredServices = ServicesDependedOn
Disposed                  Event         System.EventHandler Disposed(System.Object, System.EventArgs)
Close                     Method        void Close()
Continue                  Method        void Continue()
CreateObjRef              Method        System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                   Method        void Dispose(), void IDisposable.Dispose()
Equals                    Method        bool Equals(System.Object obj)
ExecuteCommand            Method        void ExecuteCommand(int command)
GetHashCode               Method        int GetHashCode()
GetLifetimeService        Method        System.Object GetLifetimeService()
GetType                   Method        type GetType()
InitializeLifetimeService Method        System.Object InitializeLifetimeService()
Pause                     Method        void Pause()
Refresh                   Method        void Refresh()
Start                     Method        void Start(), void Start(string[] args)
Stop                      Method        void Stop()
WaitForStatus             Method        void WaitForStatus(System.ServiceProcess.ServiceControllerStatus desiredStat...
CanPauseAndContinue       Property      bool CanPauseAndContinue {get;}
CanShutdown               Property      bool CanShutdown {get;}
CanStop                   Property      bool CanStop {get;}
Container                 Property      System.ComponentModel.IContainer Container {get;}
DependentServices         Property      System.ServiceProcess.ServiceController[] DependentServices {get;}
DisplayName               Property      string DisplayName {get;set;}
MachineName               Property      string MachineName {get;set;}
ServiceHandle             Property      System.Runtime.InteropServices.SafeHandle ServiceHandle {get;}
ServiceName               Property      string ServiceName {get;set;}
ServicesDependedOn        Property      System.ServiceProcess.ServiceController[] ServicesDependedOn {get;}
ServiceType               Property      System.ServiceProcess.ServiceType ServiceType {get;}
Site                      Property      System.ComponentModel.ISite Site {get;set;}
Status                    Property      System.ServiceProcess.ServiceControllerStatus Status {get;}
ToString                  ScriptMethod  System.Object ToString();
```
-  If you have a complex command that is being piped between multiple cmdlets, step backwards (removing parts of the pipeline) so you can see that the output is as you expect at each stage. For example:

```
PS C:\Users\mwragg> get-service | where-object {$_.displayname -like "Windows*"} | select-object status | where-object {
$_.status -eq "Running"} | measure-object | select count

Count
-----   11


PS C:\Users\mwragg> get-service | where-object {$_.displayname -like "Windows*"} | select-object status | where-object {
$_.status -eq "Running"}

 Status
 ------Running
Running
Running
Running
Running
Running
Running
Running
Running
Running
Running


PS C:\Users\mwragg> get-service | where-object {$_.displayname -like "Windows*"}

Status   Name               DisplayName
------   ----               -----------Running  AudioEndpointBu... Windows Audio Endpoint Builder
Running  Audiosrv           Windows Audio
Running  EventLog           Windows Event Log
Running  FontCache          Windows Font Cache Service
Running  FontCache3.0.0.0   Windows Presentation Foundation Fon...
Stopped  icssvc             Windows Mobile Hotspot Service
Stopped  LicenseManager     Windows License Manager Service
Running  MpsSvc             Windows Firewall
Stopped  msiserver          Windows Installer
Stopped  SDRSVC             Windows Backup
Stopped  stisvc             Windows Image Acquisition (WIA)
Stopped  TrustedInstaller   Windows Modules Installer
Running  W32Time            Windows Time
Stopped  WbioSrvc           Windows Biometric Service
Running  Wcmsvc             Windows Connection Manager
Stopped  wcncsvc            Windows Connect Now - Config Registrar
Stopped  WcsPlugInService   Windows Color System
Stopped  WdNisSvc           Windows Defender Network Inspection...
Stopped  Wecsvc             Windows Event Collector
Stopped  WEPHOSTSVC         Windows Encryption Provider Host Se...
Stopped  WerSvc             Windows Error Reporting Service
Stopped  WinDefend          Windows Defender Service
Running  Winmgmt            Windows Management Instrumentation
Stopped  WinRM              Windows Remote Management (WS-Manag...
Stopped  WMPNetworkSvc      Windows Media Player Network Sharin...
Stopped  WpnService         Windows Push Notifications Service
Running  WSearch            Windows Search
Stopped  WSService          Windows Store Service (WSService)
Stopped  wuauserv           Windows Update
Running  wudfsvc            Windows Driver Foundation - User-mo...
```

- In the ISE, press CTRL+J to load templates for various scripting constructs such as cmdlets, loops, if else, try catch blocks and more. Note that it will drop the template in to whatever script/window you have open and wherever the cursor currently is.

# Further Topics

[Powershell Cheat Sheet](http://ramblingcookiemonster.github.io/images/Cheat-Sheets/powershell-cheat-sheet.pdf) (PDF)

> This is a more detailed Cheat Sheet similar to the Basic Cheat Sheet above that provides more detailed/advanced conventions.

Worth printing and keeping nearby when you're starting to use more advanced concepts.

[An introduction to Powershell Modules](https://www.simple-talk.com/sysadmin/powershell/an-introduction-to-powershell-modules/) (Blog Post)

> For PowerShell to provide specialised scripting, especially for administering server technologies, it can have the range of Cmdlets available to it extended by means of Snapins. From version 2 there is an easier and better method of extending PowerShell: the Module. These can be distributed with the application to be administered, and a wide range of Cmdlets are now available to the PowerShell user. PowerShell has grown up.

Modules extend Powershell with additional cmdlets and you can create your own to package together a set of functions or cmdlets you've written.
Tip: In the Powershell ISE, press CTRL+J and select cmdlet (advanced function) to load a template for creating a cmdlet that you could save as a module.

[Advanced Tools & Scripting with PowerShell 3.0 Jump Start](https://mva.microsoft.com/en-US/training-courses/advanced-tools-scripting-with-powershell-3-0-jump-start-8277#fbid=wdBipBJLtCB) (Video Series)

> Take this advanced PowerShell scripting course to find out how to turn your real time management and automation scripts into useful reusable tools and cmdlets. You’ll learn the best patterns and practices for building PowerShell scripts and maintaining tools, and you’ll pick up some special tips and tricks along the way from the architect and inventor of PowerShell, Distinguished Engineer Jeffrey Snover, and IT Pro, Jason Helmick.

Not one to start with, but once you've mastered the basics this will help guide you to developing more advanced tools and scripts with Powershell.

[Active Directory Powershell Quick Reference](http://www.jonathanmedd.net/wp-content/uploads/2009/10/ADPowerShell_QuickReference.pdf) (PDF)

> A Cheat Sheet for working with the Active Directory module for Powershell that covers various constructs.

Very useful guide for when you need to work with the Active Directory cmdlets (import-module activedirectory on a Domain Controller, or having installed AD tools).

[Powershell Slack Community](https://powershell.slack.com/) (Live Chat)

> Slack is a chat tool similar to Hipchat. Powershell.slack.com has an active community of Powershell experts that you can discuss things with and also bridges various other IRC-based Powershell channels.	

An opportunity to ask questions to a community and get a (likely) instantaneous response.
Note you need [sign up](http://slack.poshcode.org/) first, but it's automated.