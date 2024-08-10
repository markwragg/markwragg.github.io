---
title: Please stop downloading my PowerShell modules
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/global-internet.jpg"
---

There was a recent trend on Twitter/X where people were sharing whether they spent more time tweeting or coding, which you can discover for yourself through this [tool](https://shiptalkers.dev/compare). Apparently I spent 61% more time tweeting than coding (in public anyway), and while I was pondering whether or not that was a good thing, [Doug Finke](https://x.com/dfinke) came to this excellent conclusion:

<blockquote class="twitter-tweet" data-align="center" data-theme="dark"><p lang="en" dir="ltr">If no one knows what you&#39;re doing, what&#39;s the point? <a href="https://t.co/JdNcIDpzzU">pic.twitter.com/JdNcIDpzzU</a></p>&mdash; Doug Finke (@dfinke) <a href="https://twitter.com/dfinke/status/1798785849238360370?ref_src=twsrc%5Etfw">June 6, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

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

Downloads | Published Date      | Name
--------- | ------------------- | -----------------
9800      | 07/06/2016 11:43:38 | ADAudit
1584      | 19/01/2017 15:21:44 | XKCD
1283      | 25/01/2017 11:30:48 | PSHipChat
1674      | 25/01/2017 13:50:21 | SlackBot
1666      | 25/05/2017 14:43:28 | Remedy
468658    | 31/12/2017 09:41:33 | Influx
14864     | 19/03/2018 09:08:11 | Watch
24308     | 06/08/2018 09:25:13 | HashCopy
1221      | 02/10/2018 11:28:45 | MacNotify
432102    | 11/04/2019 10:52:29 | Subnet
4395      | 07/08/2019 13:52:53 | Lumos
358       | 14/01/2024 15:34:23 | AzCostTools
1675      | 07/02/2024 23:48:58 | CurrencyConverter

```powershell
($modules.downloadCount | Measure -Sum).Sum

963557
```

So this is a story all about how (my life got flipped-turned upside down, and I'd like to take a minute just sit right there while I tell you how) I'm about to become a PowerShell Gallery millionaire.

### MacNotify

- **First published:** 02 Oct 2018
- **Last updated:** 17 Mar 2024
- **Download count:** 1221

...

**Should you still use it?** 

### PSHipChat

- **First published:** 25 Jan 2017
- **Last updated:** 09 Sep 2019
- **Download count:** 1283

I first uploaded [PSHipChat](https://github.com/markwragg/Powershell-Hipchat) to GitHub in March of 2016. HipChat is Atlassians answers to Slack. I think they bundled it free with some of their other products (such as Confluence) as I worked for a few companies where it was used before Slack became more ubiquitous and now (for me at least) Teams. The PSHipChat module provides some PowerShell cmdlets for sending notifications into HipChat from your scripts by invoking its API (this is the theme of a lot of my modules, FYI). This allowed my team at the time to get visibility of when certain automated tasks were running, by just adding a few lines to those scripts to trigger notifications in HipChat.

The original blog for it is [here](https://wragg.io/send-notifications-to-hipchat-with-powershell/).

**Should you still use it?** I have no idea tbh. I haven't worked anywhere with HipChat for a number of years, so I'm not sure if it still works. The project is actually now maintained under the [AtlassianPS](https://github.com/AtlassianPS) GitHub organisation, but its not getting a lot of love there either.

### XKCD

- **First published:** 19 Jan 2017
- **Last updated:** 25 Feb 2020
- **Download count:** 1584

[XKCD](https://xkcd.com/) is a brilliant webcomic that's been going since September 2005. The [XKCD module](https://github.com/markwragg/Powershell-XKCD) just wraps some PowerShell friendly cmdlets around the XKCD API which requires no key/registration and returns JSON. This was mostly for fun, and an excuse to get my head around Parameter Sets, as I blogged about [here](https://wragg.io/create-dynamic-powershell-functions-with-parameter-sets/).

**Should you still use it?** I don't see why not! It still works. The most useful thing about it is its ability to search for a specific comic by keyword, and it builds a local cache of the comics inside the Module so subsequent searches require no calls to the API. So if you're looking for a comic on a specific topic, it's as good as Google:

```
Find-Xkcd -Query covid | FT

month  num link year news safe_title              transcript alt
-----  --- ---- ---- ---- ----------              ---------- ---
7     2333      2020      COVID Risk Chart                   First prize is a free ticket to the kissing booth.
8     2346      2020      COVID Risk Comfort Zone            I'm like a vampire, except I'm not crossing that threshold even if you invite me.
9     2355      2020      University COVID Model             I admit this is an exaggeration, since I can think of at least three parties I attended while doing my degree, and I'm probably forgetting several more.
12    2395      2020      Covid Precaution Level             It's frustrating to calibrate your precautions when there's only one kind of really definitive feedback you can get, you can only get it once, and when you do it's too late.
```

Or you can open a random comic by running `Get-Xkcd -Random -Open`. It's probably not going to get you promoted, but it might make you smile.

### Remedy

- **First published:** 25 May 2017
- **Last updated:** 10 Sep 2019
- **Download count:** 1666

...

**Should you still use it?** 

### SlackBot

- **First published:** 25 Jan 2017
- **Last updated:** 10 Sep 2019
- **Download count:** 1674

The purpose of my SlackBot module was to provide a basic demonstration of how to setup a bot for Slack using their Real-time messaging API. This kind of Bot essentially monitors one or more chat channels for messages and can then respond to certain messages directed at it. Part of the motivation for sharing this was it was really fiddly to get working at the time. I blogged about it [here](https://wragg.io/powershell-slack-bot-using-the-real-time-messaging-api/). The version in the PowerShell Gallery (and on GitHub) is just the basic framework that you need to extend with whatever functionality you want the Bot to do. At the company I was working for at the time our internal version of it had that additional functionality so you could use to query for assets / tickets / changes etc. and it would make calls to other systems to retrieve the data. 

**Should you still use it?** Maybe. If you'd rather build something up yourself, I think its still a good starting point, although I am also not a big Slack user these days so I don't know entirely if it's still compatible with the API. There is a more featured alternative I can recommend though (that I suspect is also more up to date) and easier to customise/extend with additonal functionality: [PoshBot](https://github.com/poshbotio/PoshBot) by [Brandon Olin](https://github.com/devblackops).

### CurrencyConverter

- **First published:** 07 Feb 2024
- **Last updated:** 18 May 2024
- **Download count:** 1675

CurrencyConverter is the newest kid on the block, so you can somewhat forgive its low download count. 

**Should you still use it?** 

### Lumos

- **First published:** 07 Aug 2019
- **Last updated:** 07 Dec 2021
- **Download count:** 4395

...

**Should you still use it?** 

### ADAudit

- **First published:** 07 Jun 2016
- **Last updated:** 25 Feb 2020
- **Download count:** 9800

I always thought XKCD was the first module I ever published, but apparently it was [ADAudit](https://github.com/markwragg/Test-ActiveDirectory). This module actually just built on some excellent work by [Irwin Strachan](https://x.com/IrwinStrachan), who was very supportive of my work and continues to be a wonderfully generous member of the PowerShell community. ADAudit is just a set of Pester tests which validate whether an Active Directory forest is in good health. The idea is that (assuming your AD is currently healthy) you export a "gold snapshot" of its current state and then can run the tests routinely to see if anything has changed / is out of order. 

If you want to read more about it the original blog is [here](https://wragg.io/testing-active-directory-with-pester-and-powershell/).

**Should you still use it?** I would say probably not, unless you fancy bringing it up to date. I haven't updated it for several years and so its currently designed for use with Pester v3 (which is pretty old). A good alternative looks to be [Testimo](https://github.com/EvotecIT/Testimo) by EvotecIT, although I haven't personally used it, it looks to be more featured and recently maintained.

### Watch

- **First published:** 19 Mar 2018
- **Last updated:** 08 Mar 2023
- **Download count:** 14865

...

**Should you still use it?** 

### HashCopy

- **First published:** 06 Aug 2018
- **Last updated:** 02 Jul 2023
- **Download count:** 24309

...

**Should you still use it?** 

### Subnet

- **First published:** 11 Apr 2019
- **Last updated:** 10 Sep 2019
- **Download count:** 432107

...

**Should you still use it?** 

### Influx

- **First published:** 31 Dec 2017
- **Last updated:** 06 May 2023
- **Download count:** 468659

...

**Should you still use it?** 

### XKCD

- **First published:** 19th Jan 2017
- **Last updated:**  25th Feb 2020
- **Download count:** 1584

...

**Should you still use it?** 
