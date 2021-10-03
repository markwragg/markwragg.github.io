---
layout: post
title: Powershell 101
---

Built-in commands to discover cmdlets and how they work:

- `Get-Help` 

Why doesn't this work? `get-service | get-help`
Why does this work? `gcm get-service | get-help`

 - update-help
- `Get-Command`
- `Get-Member`

Cmdlets

- Parameters
- Default params
- accept pipeline input
- Tab complete cmdlet names and parameter names Get-EventL [tab] -Lo [tab] App [tab]
- -whatif and -confirm

Measure-object (measure)
Sort-Object (sort)

Use any cmd commands. Note also that some cmd command names are aliased to powershell equivalents: cd, dir (as well as unix equivalents: ls, cat). As a result in these instances you can't use the legacy switches (e.g dir /s you'd need to use dir -recurse or get-childiten -recurse).

verb-noun structure. - This makes commands more discoverable. E.g after you know get-process, you can look up other get-* as well as other *-process with get-help.

Grouped in to modules.

Objects

- Properties

Using the pipeline

- Chaining commands


