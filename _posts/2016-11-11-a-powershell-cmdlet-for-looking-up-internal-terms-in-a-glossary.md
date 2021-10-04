---
title: A Powershell cmdlet for looking up internal terms in a glossary
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2016/11/Glossary.jpg"
date: '2016-11-11 20:05:19'
tags:
- powershell
---
I've recently started a new role and as a result I have a lot of new business terms and acronyms to get to grips with. A list of about 50 of these were handed to me on my first day which was really helpful, but when I searched around the intranet and various other sources of documentation I found there were many more.

In an effort to reduce the amount of searching I might have to do in the future, I decided to see if I could compile these sources together. This was generally pretty easy to do, because the various lists were already in a tabular format, or laid out in a consistent way where I could use things like Excel's Text to Columns function to get them consistent. At first I had a simple 2 column spreadsheet of "Term" and "Description" but then it also started to make sense to broadly categorise them (based on their various sources) which was simple to do with an additional "Category" column. Finally some of the definitions had web links to more informaton so I added a "Link" column too to optionally include this.

**When I was done, I found I had managed to compile a list of over 900 terms.** Which was ~~horrifying~~ great! But then searching Excel felt clunky and so I turned to Powershell.

I created a function called `Search-Glossary`. Initially I decided I wanted the output to look akin to the Powershell help function, with capitilised titles for each of my spreadsheets columns, but only presenting each bit of information where it exists. Pretty simple with the much maligned `write-host`:

![](/content/images/2016/11/Glossary-Powershell1-2.png)

Creating the output this way initially meant that strings wider than the console wrapped half way through words and only the first line was tabbed which didn't look great. I found a [wrapText function on Stack Exchange](http://stackoverflow.com/questions/1059663/is-there-a-way-to-wordwrap-results-of-a-powershell-cmdlet) (and modified it a little to my needs) that solved that issue, so it now moves to the next line whenever a word will go over the current width of the console.

I liked this, but it didn't look so great when the search term returned multiple matches. The standard output from `format-list` was better for this and by switching to this when there were multiple matches it was then more obvious that multiple matches had been returned:

![](/content/images/2016/11/Glossary-Powershell2-1.png)

However both of these previous options broke the pipeline. While I didn't currently have a need to push the results down the pipeline, it seemed a shame not to be able to do so in the future for the sake of some formatting. To solve this I added a `-PassThru` switch (a common parameter name on some other cmdlets) that changes the result to just the default `Return` and resultant formatting (which is `format-table`) and fixes the pipeline:

![](/content/images/2016/11/Glossary-Powershell3-1.png)

Here's the full code, free to a good home:

<script src="https://gist.github.com/markwragg/0e59148c8a6e7ac1fa0c669a63584474.js"></script>

There was a certain pointless novelty to this, but it is useful to me (although still a little clunky to use as I have to open Powershell, bypass the execution policy and load my module before I can use it) however I can't see others in my team leaping to adopt it as a way to query a glossary. But the contents of that glossary are likely useful (particularly to other new starters). Which got me thinking..

This is the perfect excuse to play with something I've been meaning to try for a while: Powershell Azure Functions -- and more specifically -- leveraging them to create a slash command in Slack. See my next post for details on that (coming soon)!
