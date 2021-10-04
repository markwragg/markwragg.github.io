---
title: A PowerShell Tool Scorecard
header:
  overlay_image: "/content/images/2017/03/CheckList.jpg"
date: '2018-01-17 13:38:30'
tags:
- powershell
---
This post contains a PowerShell tool-making scorecard: a series of short questions to assess whether your custom cmdlet/function/tool is following some (generally considered) best practice design choices.

> By "tool-making" I am referring the concept of creating one or more PowerShell functions that are intended to be used by end users in the same way as any other built-in (or 3rd party) cmdlet that you might use. Some of the questions below don't necessarily apply for private/helper type functions that can (and often should) be much more simplistic.

The idea for this scorecard was inspired by several others i've seen, notably the [Joel Test](https://www.joelonsoftware.com/2000/08/09/the-joel-test-12-steps-to-better-code/) which is a scorecard for Developers to evaluate the maturity of a workplace, as well as the [Operations Report Card](https://gist.github.com/markwragg/c6542459244e02ce231d3e58d5d31284) which similarly can be used to assess the operational sophistication of an Ops team.

I also recently read the excellent [PowerShell Toolmaking in a Month of Lunches](https://www.amazon.co.uk/gp/product/1617291161/ref=as_li_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=1617291161&linkCode=as2&tag=exsite0a-21&linkId=a9836e9a1ffc6c11df0158003580775c) book which recommends following a number of these best  practices. There's also a great list on [this blog post](https://blogs.technet.microsoft.com/pstips/2014/06/17/powershell-scripting-best-practices/) which while a few years old is still very relevant. If you want a very detailed guide I recommend reading the [The PowerShell Best Practices and Style Guide in GitHub](https://github.com/PoshCode/PowerShellPracticeAndStyle). There are lots other sources of PowerShell best practice on the web, and the items below are far from an exhaustive list but (I hope) represent a good starting point. 

*[-- If you're unfamiliar with any of the concepts below, scroll further down as I have briefly explained them.]*

**PowerShell Tool Scorecard (+1 point each):**

> 1. Have you declared `[cmdletbinding()]`?
- Does your function include comment-based or external help?
- Are your input parameters declared as specific types?
- Are any mandatory parameters declared as such?
- Do your parameters have sensible defaults? (if possible)
- Does your function do just one thing?
- Does your function include `write-verbose` (or `write-debug`) statements? (if needed)
- Do you [filter left](https://technet.microsoft.com/en-us/library/2009.09.windowspowershell.aspx)? (where applicable)
- Does your function support `-whatif` and `-confirm` for any code that modifies something?
- Is your function named `verb-noun` and using an [approved verb](https://msdn.microsoft.com/en-us/library/ms714428(v=vs.85).aspx)?

> Score: __ / 10

**Bonus Round (+5 points each):**

> 1. Are there [Pester](https://github.com/pester/Pester) (or other testing framework) tests?
- Is the code coverage 100%?
- Does a check with [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer) pass?
- Is the code checked in to in a Source Control system?
- Are you using a CI pipeline to test commits?
- Are you publishing your script/module to the [PowerShell Gallery](https://www.powershellgallery.com/) (or other package repository)?
- Is the module automatically published after a successful run of the CI pipeline?
- Is there a readme.md, Wiki and/or other documentation?

> Score: __ / 40

**Penalty Points (-5 points each):**

> 1. Does your function use `Write-Host` or a `Format-*` cmdlet where it should instead output an object?
- Are you using cmdlet aliases?
- Are there any non-meaningful variable names? e.g `$s` `$c`
- Do you use the `Return` keyword?
- Have you disabled/suppressed standard errors? e.g with `$ErrorActionPreference = "SilentlyContinue"` or via `-ErrorAction`

> Score: __ / -25

---
> **Final Score: __ / 50**

---In case any of the concepts above are unfamiliar to you below are explanations of them and why they are considered important:

**1. Have you declared `[cmdletbinding()]`?**

```
[cmdletbinding()]
Param( .. )
```

By adding `[cmdletbinding()]` above the `Param()` block of a Function you gain a set of default functionality: the ability to add Write-Verbose statements to output helpful text when `-Verbose` is used, the same for `-Debug` as well as the ability to add `-WhatIf` and `-Confirm` functionality for functions that change the state of something. Even if you don't use any of these things, your function also gets the `-ErrorAction` and `-ErrorVariable` parameters which give the end user options for how they handle any errors your Function generates. There's no good reason I can think of to not use `[cmdletbinding()]` on any function that an end user interacts with.

**2. Does your function include comment-based or external help?**

```
Function Do-Something { ..
  <#
  .SYNOPSIS
      A function that does something.
  .EXAMPLE
      'Something' | Do-Something
  #>
    
```


It's really easy to add help output to a PowerShell function via a block of comments at the top of the function. Not only is this helpful to the end user who might use `Get-Help yourfunction` to work out how it works, but for anyone modifying your code in the future it gives them an excellent intro to what the function does and how it works.

**3. Are your input parameters declared as specific types?**

```
[datetime]
$Date
```

This is just a very easy way to do some input validation. There might occasionally be a scenario where you want a parameter to handle any type of input and then convert it to the relevant object in the Function and thats ok. You could probably though still declare that Parameter as a `[object]` type so that it's explicit that is what you are doing.

**4. Are any mandatory parameters declared as such?**

```
[parameter(Mandatory)]
$InputObject
```

If your function is just going to fail if one or more parameters aren't provided then it should inform the user gracefully of that before it even starts. Adding `[Parameter(Mandatory=$true)]` or (my preference) `[Parameter(Mandatory)]` before a mandatory parameter allows you to do that and let PowerShell handle the work of informing the user what they've done wrong.

**5. Do your parameters have sensible defaults? (if possible)**

```
[ipaddress]
$IPAddress = '127.0.0.1'
```

If there are some sensible default values for your parameters then you should set them within the `Param` block and ensure that you don't make those parameters mandatory (which is unnecessary and would render your default irrelevant). Sensible defaults make your tool easier to use, but make sure you expose those default values via the help text for those parameters so that it's transparent to the end user.

**6. Does your function do just one thing?**

This was a core concept covered in the Toolmaking book. As far as I interpret it, it doesn't suggest that your function can't do multiple transformations, but the idea is that your function should perform one distinct action. For example `Get-Service` only returns Service objects, it doesn't also allow a user to modify those services. That's what `Set-Service` is for. It might be that `Get-Service` queries different sources to get the properties of the service objects, but ultimately its job is to get services so that is all it does.

**7. Does your function include `write-verbose` (or `write-debug`) statements? (if needed)**

Per #1, cmdletbinding gives you the option to use `Write-Verbose` and/or `Write-Debug` to show additional output or to halt the script at key points when the end user uses the equivalent parameters on your command. In particular, `Write-Verbose` is preferable to anywhere you might have put in a single line comment in your code to explain a section. Using `Write-Verbose` still has the benefit of adding those explanations but it also exposes them to an end user when they choose to see them.

**8. Do you [filter left](https://technet.microsoft.com/en-us/library/2009.09.windowspowershell.aspx)? (where applicable)**

Anywhere you are filtering the result of a cmdlet you should do it as far left as possible. For example if i'm using `Get-ChildItem` and I want to return only .ps1 scripts then I should use the filtering parameter of that cmdlet. Or if I want to get services where the displayname property contains certain text and then iterate over those services, the `Where-Object` cmdlet that filters by displayname should occur directly after the `Get-Service` cmdlet, not further down the pipeline. Doing so will improve the performance of your script/function.

**9. Does your function support `-whatif` and `-confirm` for any code that modifies something?**

```
[cmdletbinding(SupportsShouldProcess)]
Param( .. )

..

if ($PSCmdlet.ShouldProcess('something')) { .. }
```

Again per #1, this feature is made possible by adding cmdletbinding to your script. If your script makes changes to something, you should ensure that you've added the relevant code that stops those changes from occurring (or prompts a user) if they use `-whatif` or `-confirm`. This is done by adding a `If ($PSCmdlet.ShouldProcess("Message"))` block around that part of your function that checks if these parameters have been used. Note that this should be around just the smallest part of your script possible that makes a modification, so that other (non target modifying) code still executes.

**10. Is your function named `verb-noun` and using an [approved verb](https://msdn.microsoft.com/en-us/library/ms714428(v=vs.85).aspx)?**

Ensuring that you've followed the PowerShell standard of naming your function using a `Verb-Noun` structure and using one of the [approved verbs](https://msdn.microsoft.com/en-us/library/ms714428(v=vs.85).aspx) will help make your cmdlet more consistent with the standard set of PowerShell cmdlets as well as others following best practice on the web.

![PowerShell Core now includes a description column output by Get-Verb](/content/images/2018/01/PowerShell-Core-Get-Verb.jpg)

 This makes your function easier to understand and easier to discover. Equally you should try and follow existing PowerShell practice when naming your parameters. For example generally PowerShell uses `-ComputerName` when a cmdlet can be targeted at a remote machine, you should do the same vs something like `-Host` or `-Server`.

### Bonus Questions

**1. Are there [Pester](https://github.com/pester/Pester) (or other testing framework) tests?**

[Pester](https://github.com/pester/Pester) is a fantastic tool for writing tests for your code to ensure they perform the way your expect. You should consider writing both unit tests (which test each part of your code) and integration tests (which test how the code actually performs, ideally in some sort of test environment/container). At a minimum unit tests should be used and you can do so even where you don't have access to any dependencies by using mocking.

**2. Is the code coverage 100%?**

Pester has a `-CodeCoverage` switch that allows you to see exactly what part of your code is covered by your existing tests. Getting this to 100% still isn't a guarantee that your code doesn't have bugs but its better than anything less.

**3. Does a check with [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer) pass?**

Using ScriptAnalyzer to evaluate your code helps enforce a number of these best practices. It also checks your code for dangerous behaviour, such as handling credentials as plain text instead of via a credential object.

**4. Is the code checked in to in a Source Control system?**

Even if you are the only person using the code, having it in a source control system means that you have version control and the ability to step backwards if things go wonky. If you aren't the only person using the code, source control allows others to review it and contribute to it.

**5. Are you using a CI pipeline to test commits?**

As well as writing Pester (or some other framework) tests for your code you should make sure these tests are run every time the code changes. A CI pipeline (e.g with something like AppVeyor) can automate this process. I also use a CI pipeline to execute ScriptAnalyzer on every commit.

**6. Are you publishing your script/module to the [PowerShell Gallery](https://www.powershellgallery.com/) (or other package repository)?**

If you've written a tool that others might find useful or you want to distribute it (publicly) do so via the PowerShell Gallery so that installing your tool can be done simply via `Install-Module` or `Install-Script`. If you are working on private code, a private package management repository is probably still a good idea to manage distribution and production versions of your code.

**7. Is the module automatically published after a successful run of the CI pipeline?**

Per the above, a CI pipeline should also publich the module to the appropriate package management tool so that this isn't a manual process. This should only occur if all your tests have passed.

**8. Is there a readme.md, Wiki and/or other documentation?**

If you're using GitHub, your project should have a `readme.md` that explains what the project is about, how to install it and some basic intro for how it works. If the project is of significant complexity a wiki or other documentation might be justifiable.

### Penalty Points

*-- Disclaimer: There's always an exception to a rule and no one is perfect.*

**1. Does your function use `Write-Host` or a `Format-*` cmdlet where it should instead output a usable object?**

Generally (but not exclusively) using `Write-Host` is considered bad practice because it sends output to the console rather than returning an object that the end user can manipulate or send on to another cmdlet. Equally making use of the `Format` cmdlets in your function gives the user no options for how they handle your output. Instead you should return an object and let them pipe it to a `Format` cmdlet if that's what they want. Or use the format XML feature to define how output should appear by default while still returning it as an object.

**2. Are you using cmdlet aliases?**

![](/content/images/2018/01/PowerShell-Core-Get-Alias.jpg)

PSScriptAnalyzer will complain if you do this also (unless you suppress the relevant rule). Its considered an anti-pattern because aliases are less explicit to anyone reading your code. Generally you should only use aliases at the console to shorten your typing, but when writing a script you should always use the full cmdlet names.

**3. Are there any non-meaningful variable names? e.g `$s` `$c`**

Your variable names should be descriptive regarding their purpose. Non descriptive names are unhelpful to anyone else reading your code.

**4. Do you use the `Return` keyword?**

The return keyword is generally not needed and probably doesn't do what you think it does.

**5. Have you disabled/suppressed standard errors? e.g with `$ErrorActionPreference = "SilentlyContinue"` or via `-ErrorAction`**

You ideally shouldn't suppress the standard error output of the cmdlets you use in your function. By all means handle them via `Try..Catch` but its generally considered a bad idea to hide error states from end users.

---
If you disagree with any of the above (or spot any inaccuracies) please let me know via the comments below.


