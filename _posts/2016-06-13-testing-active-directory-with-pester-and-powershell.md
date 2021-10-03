---
layout: post
title: Testing Active Directory with Pester and Powershell
image: "/content/images/2016/06/Compliance-Checkbox.jpg"
date: '2016-06-13 08:53:56'
tags:
- pester
- powershell
---

[Irwin Strachan](https://twitter.com/IrwinStrachan) published a Pester script for [Operational Testing of Active Directory](https://pshirwin.wordpress.com/2016/04/08/active-directory-operations-test/) back in April which I was keen to try out. Afterwards I extended the script to add some additional health checks of Active Directory and this post explains how the resultant combination of our work can be used to validate your Active Directory.

You can find my version of this tool here: 

> https://github.com/markwragg/Test-ActiveDirectory

Or get it from the Powershell Gallery via:

> `Install-Module ADAudit` (to grab the code to view it before installing you can use `Save-Module ADAudit` instead).

This was my second outing with Pester (previously I authored a [Pester script for my Hipchat module](https://github.com/markwragg/Powershell-Hipchat/blob/master/hipchat.Tests.ps1) which did some basic functionality tests) and it's a fantastic framework for building both unit tests of your scripts to ensure they are functioning correctly as well as performing operational testing. If you're not familiar with Pester [check out the wiki](https://github.com/pester/Pester/wiki) to help you get started.

*-- If you have Windows Management Framework 5 installed (or are running Windows 10) you may already have Pester (or you can easily install it with `Install-Module Pester`). Prior to that you can [download it from Github](https://github.com/pester/Pester/) and install it as a module.*

## Testing and validating your Active Directory

Irwin's script was a follow up to his earlier post which [captures your Active Directory configuration and stores it as an XML file](https://pshirwin.wordpress.com/2016/03/25/active-directory-configuration-snapshot/). He then realised this could be leveraged to perform Pester tests against a later capture of the configuration to determine if any drift had occurred.

After you have [downloaded the scripts from Git](https://github.com/markwragg/Test-ActiveDirectory) or [PSGallery](https://www.powershellgallery.com/packages/ADAudit/1.1), perform the following:

**1. Execute: 'Get-ADConfig.ps1'**

You need to download and execute [Get-ADConfig.ps1](https://github.com/markwragg/Test-ActiveDirectory/blob/master/Get-ADConfig.ps1) within your domain with suitable rights to create an XML file of your current Active Directory configuration.  You should do this one time initially at a point where you are satisfied that the configuration of AD is correct, then store that file as **ADGoldConfig-{date}.xml** to be used for future comparisons. 

*-- When you later use the Pester script, If you don't specify a path to this file as a parameter it by default looks for the latest version in the directory it is run:*

```language-powershell
[CmdletBinding()]
Param(
    [string]$ADFile,
    [string]$ADGoldFile = $(Get-ChildItem ("ADGoldConfig-*.xml") | Select name -last 1).name
)
```
You can run `Get-ADConfig.ps1` again manually at a later date to generate a snapshot of the Active Directory configuration for comparison. Store this in the same location as the Pester script and it will be used as the comparative config (or specify the path of a config explicitly via the `ADFile` param per the above). If one is not found by (or provided to) Pester, it will try to run Get-ADConfig.ps1 for you to generate one on the fly. 

**2. Having generated a Gold configuration and (optionally) a current configuration XML file you now execute: 'ActiveDirectory.tests.ps1', but do so by running 'invoke-pester'**

This is the Pester script and you should execute it by changing to the directory that it's in and then running `invoke-pester`. This command will look for any script in the current directory named *.tests.ps1 and execute them. While you can run the PS1 file directly, executing it with `invoke-pester` gives you a summary count at the end of tests passed/failed/skipped and allows you to optionally use other parameters such as storing the result as an XML file, or using tags to skip include or exclude certain tests.

The Pester script performs the following tests:

**Active Directory configuration checks:**

A check of the config between what has been stored previously and what has been captured now to detect configuration drift. This is almost entirely the set of tests Irwin defined in his posts.

*-- Beware there is currently a limitation in this comparison in that it doesn't validate that sets are the same. E.g if a Domain Controller is added or removed it may not trigger a fail. [Irwin explores this issue in a later post](https://pshirwin.wordpress.com/2016/04/29/operational-readiness-validation-gotchas/) and I recommend you check it out. I may modify the script in the future to account for this limitation.*

**Active Directory health checks:**

These are the tests that I added, intended to perform operational healthchecks of Active Directory in a similar way as someone might do manually. Some of these tests leverage legacy command-line tools that have been around for many years for diagnosing AD. As they are not Powershell cmdlets their results are generally not analysable as objects, so Pester tests these by looking for the presence or exclusion of certain string values.

These tests are as follows:

1. Executes `NLTest /Query` and checks the output for "Success"
- Executes `DCDiag -a` and checks the output doesn't include any mention of "failed"
- Executes `RepAdmin /showrepl /CSV` and checks each replication to ensure the "Number of Failures" count is 0
- Performs a Ping (`test-connection`) against each listed Domain Controller (as known via the configuration file)
- Uses `test-netconnection` (may require Server 2012 R2 or newer and WMF5 to work) to confirm that the following common Active Directory related TCP ports respond locally: 
 - 53, 88, 135, 139, 389, 445, 464, 636, 3268, 3269, 9389
- Checks that common Active Directory related services are running:
 - "ADWS", "BITS", "CertPropSvc", "CryptSvc", "Dfs", "DFSR", "DNS", "Dnscache", "eventlog", "gpsvc", "kdc", "LanmanServer", "LanmanWorkstation", "Netlogon", "NTDS", "NtFrs", "RpcEptMapper", "RpcSs", "SamSs", "W32Time"

## Conclusion

![](/content/images/2016/06/Pester-AD-Test-Outcome.png)

Pester will conclude indicating how many tests passed and failed. The number of tests will vary depending on the complexity of your Active Directory but in my environment 344 tests were performed in about 4 minutes, which is outstanding when you consider the effort that would be required to make the same evaluation manually.

If you have any ideas for extending these tests further, [Pull requests are welcomed](https://github.com/markwragg/Test-ActiveDirectory/pulls).