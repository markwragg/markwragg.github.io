---
title: How I became a (PowerShell) millionaire
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/diamond.jpg"
  teaser: "/content/images/2024/diamond.jpg"
date: '2024-08-13 10:00:00'
tags:
- powershell
- module
- github
---

There was a recent trend on Twitter/X where people were sharing whether they spent more time tweeting or coding, which you can discover for yourself through this [tool](https://shiptalkers.dev/compare). Apparently I spent 61% more time tweeting than coding (in public anyway), and while I was pondering whether or not that was a good thing, [Doug Finke](https://x.com/dfinke) came to this excellent conclusion:

<blockquote class="twitter-tweet" data-align="center" data-theme="dark"><p lang="en" dir="ltr">If no one knows what you&#39;re doing, what&#39;s the point? <a href="https://t.co/JdNcIDpzzU">pic.twitter.com/JdNcIDpzzU</a></p>&mdash; Doug Finke (@dfinke) <a href="https://twitter.com/dfinke/status/1798785849238360370?ref_src=twsrc%5Etfw">June 6, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Which got me thinking about the modules I've published in [GitHub](https://github.com/markwragg) and the [PowerShell Gallery](https://www.powershellgallery.com/profiles/markwragg) over the last few years (almost 10 in fact). So I've decided to revisit them, why they exist, and whether they are still useful, in case it inspires anyone else to go on their own decade long journey of publishing things to the gallery.

> If you've never published a module to the PowerShell Gallery and are interested in how to get started, there's a detailed guide in the [official documentation here](https://learn.microsoft.com/en-us/powershell/gallery/how-to/publishing-packages/publishing-a-package?view=powershellget-3.x).

I thought it might be interesting to do that in order of popularity, and (if you've used a consistent author name) it's actually quite easy (although not entirely quick as I think it has to parse every module) to get your modules from the gallery along with their download counts by using `Find-Module`. For example:

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
      Downloads          = $DownloadCount
      Description        = $Module.Description
      Link               = $Module.ProjectUri
      Published          = $FirstPublishedDate
      LastUpdated        = $Module.PublishedDate
    }
  }
}
```

Here's where I discovered a horrifying statistic: 

# Modules I've published in the PowerShell Gallery have been downloaded (in total) _almost a million times_. 

(well.. 963,850 times to be exact):

```powershell
$Modules = Find-ModuleByAuthor -Author 'Mark Wragg'
$Modules | Select Downloads,Published,Name,Link | Sort Downloads
```

Downloads     | Published           | Name              | Link                                                                                                            
------------- | ------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------
359           | 14/01/2024 15:34:23 | AzCostTools       | [https://github.com/markwragg/PowerShell-AzCostTools](https://github.com/markwragg/PowerShell-AzCostTools)            
1221          | 02/10/2018 11:28:45 | MacNotify         | [https://github.com/markwragg/PowerShell-MacNotify](https://github.com/markwragg/PowerShell-MacNotify)
1283          | 25/01/2017 11:30:48 | PSHipChat         | [https://github.com/markwragg/Powershell-Hipchat](https://github.com/markwragg/Powershell-Hipchat)                    
1584          | 19/01/2017 15:21:44 | XKCD              | [https://github.com/markwragg/Powershell-XKCD](https://github.com/markwragg/Powershell-XKCD)                          
1666          | 25/05/2017 14:43:28 | Remedy            | [https://github.com/markwragg/Powershell-Remedy](https://github.com/markwragg/Powershell-Remedy)                      
1674          | 25/01/2017 13:50:21 | SlackBot          | [https://github.com/markwragg/Powershell-SlackBot](https://github.com/markwragg/Powershell-SlackBot)                  
1675          | 07/02/2024 23:48:58 | CurrencyConverter | [https://github.com/markwragg/PowerShell-CurrencyConverter](https://github.com/markwragg/PowerShell-CurrencyConverter)
4395          | 07/08/2019 13:52:53 | Lumos             | [https://github.com/markwragg/powershell-lumos](https://github.com/markwragg/powershell-lumos)                        
9800          | 07/06/2016 11:43:38 | ADAudit           | [https://github.com/markwragg/Test-ActiveDirectory](https://github.com/markwragg/Test-ActiveDirectory)                
14898         | 19/03/2018 09:08:11 | Watch             | [https://github.com/markwragg/Powershell-Watch](https://github.com/markwragg/Powershell-Watch)                        
24316         | 06/08/2018 09:25:13 | HashCopy          | [https://github.com/markwragg/Powershell-HashCopy](https://github.com/markwragg/Powershell-HashCopy)                  
432312        | 11/04/2019 10:52:29 | Subnet            | [https://github.com/markwragg/PowerShell-Subnet](https://github.com/markwragg/PowerShell-Subnet)                      
468667        | 31/12/2017 09:41:33 | Influx            | [https://github.com/markwragg/Powershell-Influx](https://github.com/markwragg/Powershell-Influx)                      


```powershell
($modules.downloadCount | Measure -Sum).Sum

963850
```

So this is a story all about how (my life got flipped-turned upside down, and I'd like to take a minute just sit right there while I tell you how) I'm about to become a PowerShell Gallery millionaire.

### AzCostTools

- **First published:** 14 Jan 2024
- **Last updated:** 02 Jun 2024
- **Download count:** 359

AzCostTools is one of my newest projects. I introduced it in [this blog post](https://wragg.io/monitor-and-manage-your-azure-cloud-costs-with-powershell/) which was written for [Azure Spring Clean](https://www.azurespringclean.com/). It allows you to report on monthly costs across multiple Azure Subscriptions and compare those costs to one or more previous months. It also generates some simple charts using another module called [PSparklines](https://github.com/endowdly/PSparklines). The idea is to have a single command you can run once a month (or schedule) to generate insight into how your costs are changing in Azure.

**Should you use it?** Yes! (if you're using Azure and want to better understand or report on your costs), although currently it doesn't work for CSP subscriptions (where billing is done elsewhere). However I am working on a solution to this, as you can usually schedule CSP costs to be exported to a storage account and then AzCostTools will be able to import those exports.

### MacNotify

- **First published:** 02 Oct 2018
- **Last updated:** 17 Mar 2024
- **Download count:** 1221

My second least popular module is an apparently little known tool called [MacNotify](https://github.com/markwragg/PowerShell-MacNotify) that I published in 2018. This is for MacOS users of PowerShell, made possible since the introduction of PowerShell Core. MacNotify allows you to generate your own custom MacOS alerts (those little pop up boxes that appear in the top right). If you've ever heard of [BurntToast](https://github.com/Windos/BurntToast) by [Josh King](https://x.com/WindosNZ) (which has only been downloaded 19 million times), MacNotify is the Mac equivalent. In fact Josh went on to create a cross-platform module called [PoshNotify](https://github.com/Windos/PoshNotify) which wraps BurntToast, MacNotify and [PSNotifySend](https://github.com/TylerLeonhardt/PSNotifySend) (for generating notifications under Linux) so that you can use consistent cmdlet names to generate notifications on any of those operating systems.
 
**Should you use it?** Yes! In fact I made some updates to it a few months ago as I was clearing down old issues on my GitHub repos and as part of that I made sure it still works on the current version of MacOS (which it does!) so if you're running PowerShell on MacOS and want an easy way to surface events MacNotify could be what you're looking for. Or use PoshNotify and make your script OS notifications platform agnostic.

### PSHipChat

- **First published:** 25 Jan 2017
- **Last updated:** 09 Sep 2019
- **Download count:** 1283

I first uploaded [PSHipChat](https://github.com/markwragg/Powershell-Hipchat) to GitHub in March of 2016. HipChat is (was?) Atlassian's answers to Slack. I think they bundled it free with some of their other products (such as Confluence) as I worked for a few companies where it was used before Slack became more ubiquitous and now (for me at least) Teams. The PSHipChat module provides some PowerShell cmdlets for sending notifications into HipChat from your scripts by invoking its API (spoiler alert: wrapping an API is the theme of a lot of my modules, FYI). This allowed my team at the time to get visibility of when certain automated tasks were running, by just adding a few lines to those scripts to trigger notifications in HipChat.

The original blog for it is [here](https://wragg.io/send-notifications-to-hipchat-with-powershell/).

**Should you use it?** I have no idea tbh. I haven't worked anywhere with HipChat for a number of years, so I'm not sure if it still works. The project is actually now maintained under the [AtlassianPS](https://github.com/AtlassianPS) GitHub organisation, but its not getting a lot of love there either.

### XKCD

- **First published:** 19 Jan 2017
- **Last updated:** 25 Feb 2020
- **Download count:** 1584

[XKCD](https://xkcd.com/) is a webcomic of romance, sarcasm, math, and language that's been going since September 2005. The [XKCD module](https://github.com/markwragg/Powershell-XKCD) just wraps some PowerShell friendly cmdlets around the XKCD API which requires no key/registration and returns JSON. This was mostly for fun, and an excuse to get my head around [creating advanced functions with Parameter Sets](https://wragg.io/create-dynamic-powershell-functions-with-parameter-sets/).

**Should you use it?** I don't see why not! It still works. The most useful thing about it is its ability to search for a specific comic by keyword, and it builds a local cache of the comics inside the Module so subsequent searches require no calls to the API. So if you're looking for a comic on a specific topic, it's as good as Google:

```
Find-Xkcd -Query covid | FT

month  num link year news safe_title              transcript alt
-----  --- ---- ---- ---- ----------              ---------- ---
7     2333      2020      COVID Risk Chart                   First prize is a free ticket to the kissing booth.
8     2346      2020      COVID Risk Comfort Zone            I'm like a vampire, except I'm not crossing that threshold even if you invite me.
9     2355      2020      University COVID Model             I admit this is an exaggeration, since I can think of at least three parties I attended while doing my degree, and I'm probably forgetting several more.
12    2395      2020      Covid Precaution Level             It's frustrating to calibrate your precautions when there's only one kind of really definitive feedback you can get, you can only get it once, and when you do it's too late.
```

Or you can open a random comic by running `Get-Xkcd -Random -Open`.

[![xkcd starwatching comic #428](/content/images/2024/xkcd-blog.png)](https://xkcd.com/428/){: .align-center}

### Remedy

- **First published:** 25 May 2017
- **Last updated:** 10 Sep 2019
- **Download count:** 1666

My [Remedy](https://github.com/markwragg/Powershell-Remedy) module is another API wrapper, this time for the ITSM tool BMC Remedy. At the time I wrote this module I was working somewhere that used BMC Remedy and the interface for it back then was pretty clunky. Being able to pull data out of it via PowerShell allowed me to automate a lot of reporting that I needed to generate, as well as hook it up to the next module in this list (SlackBot) to allow my team to query tickets / changes / assets as part of a Slack conversation.

**Should you use it?** This is another one where I'm not sure. I haven't worked somewhere with BMC Remedy for a while. If you're using the version of Remedy I had when I wrote it (which I think was v6) then I suspect it will still work, but I doubt it's compatible with any subsequent versions.

### SlackBot

- **First published:** 25 Jan 2017
- **Last updated:** 10 Sep 2019
- **Download count:** 1674

The purpose of my [SlackBot](https://github.com/markwragg/Powershell-SlackBot) module was to provide a basic demonstration of [how to setup a bot for Slack using their Real-time messaging API](https://wragg.io/powershell-slack-bot-using-the-real-time-messaging-api/). This kind of Bot essentially monitors one or more chat channels for messages (as if it were a user) and can then respond to certain messages directed at it. Part of the motivation for sharing this code was because it was really fiddly to get working at the time (and I wanted to save others the pain).

The version in the PowerShell Gallery (and on GitHub) is just a basic framework that you need to extend with whatever functionality you want the Bot to do. At the company I was working for at the time our internal version of it had lots of additional functionality so you could use to query for assets / tickets / changes etc (as mentioned above) and it would then make calls to other systems to retrieve that data. 

**Should you use it?** Maybe. If you'd rather build something up yourself, I think its still a good starting point, although I am also not a big Slack user these days so I don't know entirely if it's still compatible with the API. There is a more featured alternative I can recommend though (that I suspect is also more up to date) and easier to customise/extend with additional functionality: [PoshBot](https://github.com/poshbotio/PoshBot) by [Brandon Olin](https://github.com/devblackops).

### CurrencyConverter

- **First published:** 07 Feb 2024
- **Last updated:** 18 May 2024
- **Download count:** 1675

[CurrencyConverter](https://github.com/markwragg/PowerShell-CurrencyConverter) is the newest module in this list. It actually began life as part of AzCostTools, where I wanted to be able to convert Azure costs into an alternative currency (as sometimes they're billed in a currency that is not your own). I realised this was probably [useful functionality in its own right](https://wragg.io/Perform-currency-conversion-with-PowerShell/) and there didn't seem to be a recent module in the gallery that could do the same. One of the appealing things about my module is that it uses a backend service that has an open API, so there's no need to register an account / generate a key. This caused some confusion on Reddit when I shared it as people thought I'd just hard coded my own key into the tool but this is not the case.

**Should you use it?** Yes! It should work fine as long as the API it wraps continues to exist. I also added crypto currency conversion recently at the request of someone who raised an issue on it's GitHub project. Beware though that for traditional currencies the exchange rates only update daily.

### Lumos

- **First published:** 07 Aug 2019
- **Last updated:** 07 Dec 2021
- **Download count:** 4395

The year was 2019. Windows 10 had just got a dark mode feature, as had MacOS, but neither offered the ability to switch between light and dark mode automatically. Hence [Lumos](https://github.com/markwragg/PowerShell-Lumos) was born, and the world was saved. I think both OSes do allow you to configure automatic switching now, so Lumos is kind of redundant. That being said it does allow you to switch Windows to dark mode, but keep apps as light mode (or vice versa) and it also allows you to configure a specific wallpaper for each mode that it can then switch for you at the same time (can Windows do that? Maybe! I haven't checked). Plus it's fun to type `Invoke-Lumos` and feel like (you're) a Wizard (Harry).

**Should you use it?** Yes, be magical.

### ADAudit

- **First published:** 07 Jun 2016
- **Last updated:** 25 Feb 2020
- **Download count:** 9800

I always thought XKCD was the first module I ever published, but apparently the title belongs to [ADAudit](https://github.com/markwragg/Test-ActiveDirectory). This module actually just built on some excellent work by [Irwin Strachan](https://x.com/IrwinStrachan), who was very supportive and continues to be a wonderful and generous member of the PowerShell community. ADAudit is just a set of Pester tests which validate whether an Active Directory forest is in good health. The idea is that (assuming your AD is currently healthy) you export a "gold snapshot" of its current state and then can run the tests routinely to see if anything has changed / is out of order. 

If you want to read more about it the original blog is [here](https://wragg.io/testing-active-directory-with-pester-and-powershell/).

**Should you use it?** I would say probably not, unless you fancy bringing it up to date. I haven't updated it for several years and so it's currently designed for use with Pester v3 (which is pretty old). A good alternative looks to be [Testimo](https://github.com/EvotecIT/Testimo) by EvotecIT, although I haven't personally used it, it looks to be more featured and recently maintained.

### Watch

- **First published:** 19 Mar 2018
- **Last updated:** 08 Mar 2023
- **Download count:** 14898

Linux has a Watch command and Windows doesn't which was frankly unacceptable. [Watch](https://github.com/markwragg/PowerShell-Watch) allows you to [run a command (or script block) continuously until the output changes](https://wragg.io/watch-for-changes-with-powershell/). It's useful if you're waiting for something to occur, or want to catch something in the act (like a process starting, or terminating). Ultimately all we're really doing here is wrapping some code in a `while` loop and comparing it's output to the previous iteration. One of the coolest things about `Watch-Command` is that you can just pipe a string of commands to it. It then gets the full pipeline that proceeded it from `$MyInvocation` so it can loop those commands and watch for changes. You can also send it a script block, but you don't have to. This makes it pretty convenient to use.

```powershell
Get-Service | Select Name,Status | Watch-Command
```

**Should you use it?** Yes! It still works as far as I know. I don't use it in earnest but every now and again I have some need to quickly monitor something for change and I remember it exists. And 14,865 other people apparently did too.

### HashCopy

- **First published:** 06 Aug 2018
- **Last updated:** 02 Jul 2023
- **Download count:** 24316

24.3K downloads (you can tell we're getting serious now, because I'm using "K"). [HashCopy](https://github.com/markwragg/PowerShell-HashCopy) came into existence because I needed to know if some files in one folder matched some files in another folder, but I couldn't trust the modified dates to tell me (long story short: Git). I discovered PowerShell has a `Get-FileHash` command, which you can use to compute the hash value of a file. So by computing the hash values of the files I wanted to compare, I could quickly determine if they were different. 

**Should you use it?** Yes! I guess I'm not the only person to face this problem based on the downloads, and I don't think much has changed for `Get-FileHash` so it should still work. You can read the original blog post [here](https://wragg.io/a-powershell-cmdlet-to-copy-files-based-on-hash-difference/).

### Subnet

- **First published:** 11 Apr 2019
- **Last updated:** 10 Sep 2019
- **Download count:** 432312

Now we take a giant leap to the last two modules that do most of the heavy lifting for this two-thumbed millionaire. If I'm being honest here, I suspect much of the success of my [Subnet](https://github.com/markwragg/PowerShell-Subnet) module is because I noticed no one had nabbed the name "Subnet". It's possibly an oversight (and a bit of a security flaw) of the PowerShell Gallery that anyone can take any available name. 

The Subnet module has 3 commands, `Get-Subnet` calculates details of a specified subnet (such as it's subnet mask, IP range and the host addresses), `Get-NetworkClass` tells you the network class of a specified IP and `Test-PrivateIP` gives you a true/false result for public/private IPs. 

**Should you use it?** Yes! I personally use it pretty frequently as it's usually more convenient than using a subnet calculator website. It's also a great tool if you need to generate subnet details for a list of IP addresses.

### Influx

- **First published:** 31 Dec 2017
- **Last updated:** 06 May 2023
- **Download count:** 468667

And finally, with a horrifying 468,659 downloads and counting is [Influx](https://github.com/markwragg/PowerShell-Influx). Guess what? I had a need to write some metrics into Influx (which is a time-series database that I heartily recommend, particularly if you pair it with Grafana), using the API directly was a little clunky, and so I did something I'd ~never~ done before and turned to PowerShell. The Influx module makes writing statistics into an Influx database pretty simple. And I guess lots of other people thought so too. 

Read the original blog post [here](https://wragg.io/windows-based-grafana-analytics-platform-via-influxdb-and-powershell/).

**Should you use it?** I mean, I guess?! One regret I have is that I bundled into the module various other commands that I wrote for retrieving values from things like VMWare and 3PAR and then writing those into Influx and I think that kind of bloats the module now, but for all I know 100k people think those things are really useful. 

---

If you've made it this far, thanks for indulging me through this walk down memory lane. You've perhaps noticed there's a consistent theme to the modules above, which is that most of them started life as a way to solve some specific problem for me, that I realised could probably also help someone else. None of the modules above are perfect, but I suspect they've saved (perhaps a lot) of people (hopefully a lot) of time, and that's worth a million. Here's to the next.

![Leonardo DiCaprio Cheers](/content/images/2024/Leonardo-Dicaprio-Cheers.jpg){: .align-center}
