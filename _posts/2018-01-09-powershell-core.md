---
title: PowerShell Core 6.0
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2018/01/PowerShell-6-Core-1.jpg"
  teaser: "/content/images/2018/01/PowerShell-6-Core-1.jpg"
date: '2018-01-09 00:01:03'
tags:
- powershell
- windows
---
PowerShell now comes in two flavours, (Vanilla) Windows PowerShell and PowerShell Core (..Rocky Road? ice cream flavour TBD). PowerShell Core is a version of PowerShell built on top of [.NET Core](https://github.com/dotnet/core). The GA version of PowerShell Core is due to be released on the 10th January and Release Candidate versions have been available for some time.

Despite being a new version of PowerShell, its release continues the existing versioning history and also marks the next major release of PowerShell. As such the release will be numbered 6.0 (even though it's effectively 1.0 for Core). The [PowerShell dev team have decided to continue the existing version numbering](https://github.com/PowerShell/PowerShell/issues/5165) on the basis that:

> *"..as an engine and platform, PSCore6 is a superset of Windows PowerShell 5.1."* -- [Steve Lee](https://twitter.com/steve_msft?lang=en)

While there are two versions today, it is important to note that PowerShell Core is being touted by Microsoft as the future of PowerShell, with the expectation that only bug fixes will be added to the (still delicious) Vanilla-flavoured PowerShell going forward. 

They've been pretty clear that **all new feature development will go in to PowerShell Core.** And as such..

>  *"..there are no plans for a Windows PowerShell 6.0."* -- [Joey Aiello](https://twitter.com/joeyaiello?lang=en)

![](/content/images/2018/01/say-what-meme.jpg)

### So.. why are there two versions?

Windows PowerShell was written on top of the .NET Framework. As a result, it can only be installed on Windows. PowerShell Core (due to being built on top of .NET Core) is cross platform, and available for install on Windows, macOS and various distributions of Linux.

**In an increasingly cloudified cross-platform world, this makes a lot of sense.**

However, while the PowerShell team want Core to ultimately have as much backwards compatibility as possible, right now there are some things that won't work in Core that do work in Vanilla. As such the existing version of PowerShell remains available as a fallback/alternative.

### What's different?

Aside from the cross-platform compatibility, here are some key differences:

- The PowerShell Core shell and desktop icon are back in black..

![](/content/images/2018/01/PowerShell-Core-6.png)

*(-- This makes sense because PSCore is a slimmed down version of PowerShell and everyone knows black is slimming.)*

- ..and the executable has changed to `pwsh.exe`. Both choices are likely to avoid confusion with Vanilla PowerShell, as (on Windows) they can be installed side by side.
- It's (for some things) fast(er). Not necessarily for every script/operation, but for example: I have a set of Pester tests for a module that take 53 seconds to run under Windows PowerShell. Under PowerShell Core they complete in a blistering 15 seconds.
- The install directory for PowerShell Core is `C:\Program Files\PowerShell` (where as it's `\WindowsPowerShell` for Vanilla) and the user directory is `C:\users\<username>\documents\PowerShell`. This is where you'll store your profile.ps1 and any user specific Modules/Scripts. As a result Windows PowerShell and PowerShell Core do not share the same `$env:PSModulePath`, which means if you run them together you will have separate versions of modules installed for each.
- As already mentioned, the underlying .NET version is different and so some existing things won't work (see: downsides).

### What's new?

The release of PowerShell Core also sees the release of the new 6.0 version of PowerShell, so while this is a significant change from an underlying engine perspective there are also a number of new features and enhancements as you'd expect with any major release. 

Here are some of my favourites:

- PowerShell remoting over SSH. This obviously makes sense with the new support for non-Windows OSes and should mean you can PowerShell remote anywhere PowerShell Core is supported/installed. SSH support is being built in to the existing `*-PSSession` cmdlets.
- `Invoke-WebRequest` now has a `-SkipCertificateCheck` switch. If you've ever struggled with the hacky workarounds of connecting to a site that is using a self-signed certificate, this switch should (hopefully) make that a problem of the past.
- It's now possible to get an array of characters via the `..` operator. For example: `'a'..'z'` (note you need to include the quote marks around the letters).
- Putting `&` at the end of a pipeline will cause the pipeline to be run as a PowerShell job (this probably won't confuse people in the future..):

![](/content/images/2018/01/PowerShell-Core-6-Ampersand.jpg)

- $OutputEncoding default has been changed to be UTF8 without BOM rather than ASCII. The default encoding output of PowerShell has been a sticking point in the [past](https://stackoverflow.com/questions/40098771/changing-powershells-default-output-encoding-to-utf-8).
- They've added `-AsHashtable` to `ConvertFrom-Json` to return a Hashtable instead. I can see that being useful.
- They've made `Import-Csv` support CR, LF and CRLF as line delimiters. Again, can see lots of future use cases for this that will avoid hacky `-split` type workarounds.

If you'd like to see all of the changes you can review the [Change Log](https://github.com/PowerShell/PowerShell/blob/master/CHANGELOG.md) on Git.

### What are the downsides?

Due to the underlying .NET engine change there are a number of things that (at present) are not supported in PowerShell Core. 

- There is no WMI support. For example there is no `Get-WMIObject` cmdlet at present in PowerShell Core. However the newer `Get-CIM*` cmdlets are present:

![](/content/images/2018/01/PowerShell-Core-6-Get-CimInstance.jpg)

- The ActiveDirectory module/cmdlets are not yet supported.
- There's no support for the Windows Presentation Foundation (WPF) or Windows Forms.
- The Workflows functionality is missing, although I think this was a rarely used feature of PowerShell for most.
- Many other first and third party modules may not be compatible and will need to be identified (and potentially updated) on a case by case basis.

There is apparently a Windows Compatibility Pack for .NET core in the works that adds back in some of the missing functionality. [Richard Siddaway blogged about this recently](https://richardspowershellblog.wordpress.com/2018/01/04/windows-compatibility-pack/).

- There is no PowerShell ISE for PowerShell Core. ISE has been sidelined for a while and Microsoft now recommend Visual Studio Code with the PowerShell plugin as the defacto PowerShell IDE.

If you're using AppVeyor and would like to test PowerShell Core support as part of your pipeline, I suggest watching [this issue on the AppVeyor site](https://help.appveyor.com/discussions/questions/16107-different-images) as [Oliver Lipkau](https://github.com/lipkau) has posed the question of the best way to implement this. I suspect when Core is GA AppVeyor will provide an official image fairly quickly.

### Final thoughts

Having browsed around various blogs and articles to source information for this post I can see that this change is going to cause some people some anguish. I'm sure it will frustrate me in the near future, but my feeling at this point is that we should get on board, the earlier the better. If you are currently maintaining any public modules or scripts you should check them for compatibility and identify the outcome in your documentation/readme.md. 

If PowerShell Core is a success as a cross-platform scripting language then that will add tremendous career value to those of us who are (traditionally lesser-paid than our Unix counterpart) Microsoft-stack professionals. It could also greatly reduce the complexity of the tools/scripts/technologies that we have to support in the future.