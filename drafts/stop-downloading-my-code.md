---
title: Please stop downloading my code
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/global-internet.jpg"
---

There was a recent trend on Twitter/X where people were sharing whether they spent more time tweeting or coding, which you can discover for yourself through this [tool](https://shiptalkers.dev/compare). Apparently I spent 61% more time tweeting than coding (in public anyway), and while I was pondering whether or not that was a good thing, [Doug Finke](https://x.com/dfinke) came to this excellent conclusion:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">If no one knows what you&#39;re doing, what&#39;s the point? <a href="https://t.co/JdNcIDpzzU">pic.twitter.com/JdNcIDpzzU</a></p>&mdash; Doug Finke (@dfinke) <a href="https://twitter.com/dfinke/status/1798785849238360370?ref_src=twsrc%5Etfw">June 6, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Which got me thinking about the modules I've published in GitHub and the PowerShell Gallery over the last few years (almost 10 years in fact). And channelling my inner Andrew Pla, I thought it might be inspiring at best (and interesting at worst) to take a trip down memory lane and write about how and I why I published those modules.

I thought it might be interesting to do that in order of popularity, and (if you've used a consistent author name) it's actually quite easy to get your modules from the Gallery along with their download counts by using `Find-Module`. For example:

```powershell
function Find-ModuleByAuthor {
  [cmdletbinding()]
  param(
    $Author = 'Mark Wragg'
  )
  
  Write-Information 'Finding all modules..'
  $Modules = Find-Module | Where-Object { $_.Author -eq $Author }
  
  foreach ($Module in $Modules) {
  
    Write-Information "Finding all versions of $($Module.Name).."
    $ModuleVersions = Find-Module -Name $Module.Name -AllVersions
  
    $DownloadCount = [int]($ModuleVersions.AdditionalMetadata.versionDownloadCount | Measure-Object -Sum).Sum
  
    [PSCustomObject]@{
      Name          = $Module.Name
      DownloadCount = $DownloadCount
      Description   = $Module.Description
      ProjectUri    = $Module.ProjectUri
      PublishedDate = $Module.PublishedDate
    }
  }
```

Here's why I discovered a horrifying statistic: modules I've published in the PowerShell Gallery have been downloaded _almost a million times_ (963,557 to be exact).