---
title: Three things you might not know about Import-module
image: "/content/images/2016/05/media-20160429-1.jpg"
date: '2016-05-01 15:00:00'
tags:
- powershell
---
In Powershell the Import-Module cmdlet allows you to extend the cmdlets available to you within your script or console. This can be official extensions, such as the activedirectory module that is included with AD Tools, or custom modules you have written yourself to group together a useful set of functions or commands. This post briefly covers a few tricks i've discovered about import-module that i've found repeatedly useful.

If you want to see which modules are available to you (and where they are located on the system) execute `Get-Module -ListAvailable`.

### 1. Re-importing a module that is already loaded does nothing, unless you use the -force

When developing your own modules, you may wish to test changes to your code by re-importing your module via repeatedly using the import-module command. What Powershell doesn't make obvious is that if a module has been previously loaded, import-module does nothing. This can be frustrating during development as the console returns no indication that it has not reloaded your module. To ensure your module does get reloaded each time, simply include the -force parameter.

`Import-module mymodule -force`

This is an alternative to executing remove-module then import-module again, or you could close and relaunch your Powershell session.

### 2. You can limit the cmdlets that are loaded by using -cmdlet

If you want to limit specifically which cmdlets are loaded when calling import-module, simply use the -cmdlet parameter and then a comma separated list of cmdlets. I find this useful when working with the ActiveDirectory module in a script where I only want one or two commands (such as get-aduser). I suspect there is a small speed benefit to specifying which cmdlets to load, but this also has the security benefit of ensuring the script is only using the cmdlet you intended.

`Import-module activedirectory -cmdlet get-aduser,get-adcomputer`

### 3.  You can use -verbose to detail the cmdlets and other items loaded from the module

If you want to see exactly what is loaded when an import-module command is used add the -verbose parameter and get verbose output to the console listing (among other things) each cmdlet that is loaded.

`Import-module activedirectory -verbose`