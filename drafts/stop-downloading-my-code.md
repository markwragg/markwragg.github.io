---
title: Please stop downloading my code (aka the Fresh Prince of PSGallery)
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/global-internet.jpg"
---

There was a recent trend on Twitter/X where people were sharing whether they spent more time tweeting or coding, which you can discover for yourself through this [tool](https://shiptalkers.dev/compare). Apparently I spent 61% more time tweeting than coding (in public anyway), and while I was pondering whether or not that was a good thing, [Doug Finke](https://x.com/dfinke) came to this excellent conclusion:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">If no one knows what you&#39;re doing, what&#39;s the point? <a href="https://t.co/JdNcIDpzzU">pic.twitter.com/JdNcIDpzzU</a></p>&mdash; Doug Finke (@dfinke) <a href="https://twitter.com/dfinke/status/1798785849238360370?ref_src=twsrc%5Etfw">June 6, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Which got me thinking about the modules I've published in GitHub and the PowerShell Gallery over the last few years (almost 10 years in fact). So I've decided to revisit them and why they exist. I thought it might be interesting to do that in order of popularity, and (if you've used a consistent author name) it's actually quite easy (although not entirely quick as I think it has to parse every module) to get your modules from the Gallery along with their download counts by using `Find-Module`. For example:

```powershell
function Find-ModuleByAuthor {
  [cmdletbinding()]
  param(
    [parameter(Mandatory)]
    $Author
  )

  Write-Verbose "Finding all modules by $Author.."
  $Modules = Find-Module | Where-Object { $_.Author -eq $Author }

  foreach ($Module in $Modules) {

    Write-Verbose "Finding all versions of $($Module.Name).."
    $ModuleVersions = Find-Module -Name $Module.Name -AllVersions

    $FirstPublishedDate = ($ModuleVersions | Sort { $_.Version -as [version] } | 
      Select -First 1).PublishedDate
    $DownloadCount = [int]($ModuleVersions.AdditionalMetadata.versionDownloadCount | 
      Measure-Object -Sum).Sum

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

# Modules I've published in the PowerShell Gallery have been downloaded _almost a million times_. 

(well.. 963,557 to be exact):

```powershell
$Modules = Find-ModuleByAuthor -Author 'Mark Wragg'
$Modules | Select Name,FirstPublishedDate,DownloadCount,ProjectUri | Sort FirstPublishedDate
```

Downloads | Published Date  | Name
------------- | ------------------- | -----------------
9800          | 07/06/2016 11:43:38 | ADAudit
1584          | 19/01/2017 15:21:44 | XKCD
1283          | 25/01/2017 11:30:48 | PSHipChat
1674          | 25/01/2017 13:50:21 | SlackBot
1666          | 25/05/2017 14:43:28 | Remedy
468658        | 31/12/2017 09:41:33 | Influx
14864         | 19/03/2018 09:08:11 | Watch
24308         | 06/08/2018 09:25:13 | HashCopy
1221          | 02/10/2018 11:28:45 | MacNotify
432102        | 11/04/2019 10:52:29 | Subnet
4395          | 07/08/2019 13:52:53 | Lumos
358           | 14/01/2024 15:34:23 | AzCostTools
1675          | 07/02/2024 23:48:58 | CurrencyConverter

```powershell
PS> $Modules.downloadCount | Measure-Object -Sum

963557
```

So this is a story all about how (my life got flipped-turned upside down, and I'd like to take a minute just sit right there while I tell you how) I'm about to become a PowerShell Gallery millionaire.

### ADAudit

I always thought XKCD was my first module, but apparently it was [ADAudit](https://github.com/markwragg/Test-ActiveDirectory). This module actually just built on some excellent work by Irwin Strachan, who continues to be a wonderfully generous member of the PowerShell community. ADAudit is just a set of Pester tests which validate whether an Active Directory forest is in good health. The idea is that (assuming your AD is currently healthy) you create a "gold snapshot" of its current state and then can run the tests routinely to see if anything has changed / is out of order. 

If you want to read more about it the original blog is here: https://wragg.io/testing-active-directory-with-pester-and-powershell/

**Should you still use it?** I would say probably not, unless you fancy bringing it up to date. I haven't updated it for several years and so its currently designed for use with Pester v3. A good alternative looks to be [Testimo](https://github.com/EvotecIT/Testimo) by EvotecIT, although I haven't personally used it, it looks to be more featured and more recently maintained.

### XKCD

..