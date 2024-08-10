---
title: Please stop downloading my code
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/global-internet.jpg"
---

There was a recent trend on Twitter/X where people were sharing whether they spent more time tweeting or coding, which you can discover for yourself through this [tool](https://shiptalkers.dev/compare). Apparently I spent 61% more time tweeting than coding (in public anyway), and while I was pondering whether or not that was a good thing, [Doug Finke](https://x.com/dfinke) came to this excellent conclusion:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">If no one knows what you&#39;re doing, what&#39;s the point? <a href="https://t.co/JdNcIDpzzU">pic.twitter.com/JdNcIDpzzU</a></p>&mdash; Doug Finke (@dfinke) <a href="https://twitter.com/dfinke/status/1798785849238360370?ref_src=twsrc%5Etfw">June 6, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Which got me thinking about the modules I've published in GitHub and the PowerShell Gallery over the last few years (almost 10 years in fact). I thought it might be interesting to do that in order of popularity, and (if you've used a consistent author name) it's actually quite easy (although not entirely quick as I think it has to parse every module) to get your modules from the Gallery along with their download counts by using `Find-Module`. For example:

```powershell
function Find-ModuleByAuthor {
  [cmdletbinding()]
  param(
    [parameter(Mandatory)]
    $Author
  )

  Write-Information 'Finding all modules..'
  $Modules = Find-Module | Where-Object { $_.Author -eq $Author }

  foreach ($Module in $Modules) {

    Write-Information "Finding all versions of $($Module.Name).."
    $ModuleVersions = Find-Module -Name $Module.Name -AllVersions

    $FirstPublishedDate = ($ModuleVersions | Sort-Object { $_.Version -as [version] } | Select -First 1).PublishedDate
    $DownloadCount = [int]($ModuleVersions.AdditionalMetadata.versionDownloadCount | Measure-Object -Sum).Sum

    [PSCustomObject]@{
      Name               = $Module.Name
      DownloadCount      = $DownloadCount
      Description        = $Module.Description
      ProjectUri         = $Module.ProjectUri
      FirstPublishedDate = $FirstPublishedDate
      LastPublishedDate  = $Module.PublishedDate
    }
  }
}
```

Here's where I discovered a horrifying statistic: 

> # Modules I've published in the PowerShell Gallery have been downloaded _almost a million times_. 

(well.. 963,557 to be exact):

```powershell
$Modules = Find-ModuleByAuthor -Author 'Mark Wragg'
$Modules | Select Name,FirstPublishedDate,DownloadCount,ProjectUri | Sort FirstPublishedDate
```
DownloadCount | FirstPublishedDate  | Name              | ProjectUri
------------- | ------------------- | ----------------- | ---------------------------------------------------------
9800          | 07/06/2016 11:43:38 | ADAudit           | https://github.com/markwragg/Test-ActiveDirectory
1584          | 19/01/2017 15:21:44 | XKCD              | https://github.com/markwragg/Powershell-XKCD
1283          | 25/01/2017 11:30:48 | PSHipChat         | https://github.com/markwragg/Powershell-Hipchat
1674          | 25/01/2017 13:50:21 | SlackBot          | https://github.com/markwragg/Powershell-SlackBot
1666          | 25/05/2017 14:43:28 | Remedy            | https://github.com/markwragg/Powershell-Remedy
468658        | 31/12/2017 09:41:33 | Influx            | https://github.com/markwragg/Powershell-Influx
14864         | 19/03/2018 09:08:11 | Watch             | https://github.com/markwragg/Powershell-Watch
24308         | 06/08/2018 09:25:13 | HashCopy          | https://github.com/markwragg/Powershell-HashCopy
1221          | 02/10/2018 11:28:45 | MacNotify         | https://github.com/markwragg/PowerShell-MacNotify
432102        | 11/04/2019 10:52:29 | Subnet            | https://github.com/markwragg/PowerShell-Subnet
4395          | 07/08/2019 13:52:53 | Lumos             | https://github.com/markwragg/powershell-lumos
358           | 14/01/2024 15:34:23 | AzCostTools       | https://github.com/markwragg/PowerShell-AzCostTools
1675          | 07/02/2024 23:48:58 | CurrencyConverter | https://github.com/markwragg/PowerShell-CurrencyConverter

```powershell
PS> $Modules.downloadCount | Measure-Object -Sum

963557
```
