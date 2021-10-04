---
title: Create dynamic PowerShell functions with Parameter Sets
header:
  overlay_image: "/content/images/2017/01/xkcd-print-crop.jpg"
date: '2017-01-18 20:00:33'
tags:
- powershell
---
While developing a PowerShell function to query the API of the webcomic [XKCD](http://xkcd.com) I decided to explore and implement Parameter Sets. These allow you to provide your users with different sets of parameters based on different use cases (assuming you have multiple use cases), which as a result provides a more dynamic set of functionality from a single cmdlet. 

You don't have to use parameter sets to support multiple functionality (your function could just ignore the use of parameters that do not apply) but doing so makes your function more explicit to the user as it ensures that it outright rejects the use of invalid parameter combinations:

![](/content/images/2017/01/Get-XKCD-Invalid-Parameters.png)

In addition, the `get-help` output for your cmdlet is automatically more explicit/helpful, as I will show later. Input validation is sometimes important and (probably) never a bad thing:

![](http://imgs.xkcd.com/comics/exploits_of_a_mom.png)

> XKCD provides an API that is free to use, returns JSON and is [described here](https://xkcd.com/json.html).

*--PowerShell is by no means the best way to read the XKCD webcomic and on it's own this function is of limited use. However my purpose for this function was to add some fun functionality to my [PowerShell based Slack Bot](http://wragg.io/powershell-slack-bot-using-the-real-time-messaging-api/). I wanted the bot to be able to pull comics directly in to a Slack Channel when requested, perhaps along with the title and image alt text that you get when viewing through [XKCD.com](http://xkcd.com), which is data that the API provides.*

## Use cases

As I developed my function, I found that I wanted to support various use cases, which (presently) are as follows:

1. Return the latest comic (default behaviour if no parameters passed) or one or more specific comics, by number: `-num`.
- Return a random comic, from the full range or within a specific range. `-random` `-min` and `-max`.
- Return the latest x number of comics `-newest`.

Additionally, I wanted the user to be able to optionally download the image of the comic: `-download` and specify a destination for the download: `-path`.

Here's how those parameters look without organising them in to sets:

```
Param (
     [switch]$Random,
     [int]$Min = 1,
     [int]$Max = (Invoke-RestMethod http://xkcd.com/info.0.json").num,        
     [int]$Newest,
     [switch]$Download, 
     [string]$Path = $PWD,
     [int[]]$Num = $Max     
)
```
As you can see, I've defined types for the parameters (which provides some simple validation that the user input is the right type) and I've specified some defaults where appropriate (including an initial web call that grabs the number of the latest comic).

## Defining my Parameter Sets

I have three use cases, which i'm naming "specific", "random" and "newest". To allocate a parameter to a set, you use `[Parameter(ParameterSetName=’YourName’)]` in front of each parameter, as follows:

```
   Param (
        [Parameter(ParameterSetName=’Random’)][switch]$Random,
        [Parameter(ParameterSetName=’Random’)][int]$Min = 1,
        [Parameter(ParameterSetName=’Random’)][int]$Max = (Invoke-RestMethod "http://xkcd.com/info.0.json").num,        
        [Parameter(ParameterSetName=’Newest’)][int]$Newest,
        [switch]$Download,
        [ValidateScript({Test-Path $_ -PathType ‘Container’})] 
        [string]$Path = $PWD,
        [Parameter(ParameterSetName=’Specific’,ValueFromPipeline=$True,Position=0)][int[]]$Num = $Max     
    )
```
You can see that I have not specified a `ParameterSetName` for `$Download` or `$Path` as I want these to be available to all sets (and as a result they are). 

*--There is an alternative way to approach this scenario, which is to define those parameters multiple times explicitly in each set. This is obviously more explicit and has the added benefit of allowing you to specify a parameter position order for every parameter in each set, but that felt overkill for my function.*

I have specified a parameter position for `$number` (`Position=0`) as it seemed logical that a user might want to say (for example) `Get-XKCD 123` without having to explicitly use the `-num` parameter name, but for all the other parameters explicit use felt more appropriate/likely.

My function has a `Process` block and respects the pipeline, so I've defined `$num` as `[int[]]` (note the extra set of square brackets) which means it accepts array input. This means I can return multiple comics at once by doing (for example) `1,2,3 | Get-XKCD` or `Get-XKCD 1..5`. 

Finally I decided to validate that when `-Path` is used, a valid folder path is provided. To do that I do this: `[ValidateScript({Test-Path $_ -PathType ‘Container’})]`.

There's plenty of further options to do more explicit parameter validation, but I felt I'd taken it far enough for my function (to balance sensible functionality against bloated code).

## Get-Help

As mentioned earlier, aside from ensuring your function rejects odd combinations of parameters, another benefit to doing this is a more explicit help output by default. Without having defined any explicit Help text, `Get-Help Get-XKCD` returns this:

![](/content/images/2017/01/Get-Help-Get-XKCD.png)

Which as you can see, defines the three different use cases under "Syntax". Pretty cool :).

## The full code

I've created a project in GitHub for this: [Powershell-XKCD](https://github.com/markwragg/Powershell-XKCD/), as i'm considering adding further functionality and other XKCD related cmdlets. Check it out on [GitHub](https://github.com/markwragg/Powershell-XKCD/blob/master/XKCD.psm1), or via the embedded the code below:

<script src="http://gist-it.appspot.com/github/markwragg/Powershell-XKCD/blob/master/XKCD.psm1"></script>

## Usage examples

Here are some usage examples as detailed in [readme.md](https://github.com/markwragg/Powershell-XKCD/blob/master/README.md).

> XKCD is a webcomic by Randall Munroe. Please respect the license of his work as described here: http://xkcd.com/license.html.

1) `Get-XKCD`
By default (and with no specified parameters) the function will return a PowerShell object with the details of the latest webcomic. For example:

```
month      : 1
num        : 1786
link       : 
year       : 2017
news       : 
safe_title : Trash
transcript : 
alt        : Plus, time's all weird in there, so most of it probably broke down and decomposed hundreds of years ago. Which reminds me, I've been meaning to get in touch 
             with Yucca Mountain to see if they're interested in a partnership.
img        : http://imgs.xkcd.com/comics/trash.png
title      : Trash
day        : 16
```

2) `Get-XKCD 1` or `Get-XKCD -num 1`
Specify the number of specifc comic/s you want to access via the -num parameter (this is a positional parameter so it doesn't need to be explicitly used).

3) `Get-XKCD -Random` or `Get-XKCD -Random -Min 1 -Max 10`
Use the -Random switch to get a Random comic. Optionally specify Min and Max if you want to restrict the randomisation to a specific range of comic numbers.

4) `Get-XKCD -Newest 5`
Use the -Newest switch to get a specified number of the newest comics. Note this cannot be used with -Random (and vice versa).

5) `Get-XKCD 1,5,10` or `10..20 | Get-XKCD`
The number paramater accepts array input and pipeline input, so you can use either to return a specific selection in one hit.

6) `Get-XKCD -Download` or `Get-XKCD 1337 -Download -Path C:\XKCD`
Use the -Download switch to download the image/s of the returned comics. Optionally specify a path to download to, by default it uses the current directory. Note you can use -Download and -Path with any of the other parameters.

7) `1..10 | % { Get-XKCD -Random -min 1 -max 100 | select num,img } | FT -AutoSize`
This calls Get-XKCD 10 times in a foreach loop, returning the number and image URL of 10 random comics from the first 100 comics and presenting them as an autosized table.

## Further reading

There's lots of other great articles online about Parameter Sets as well as other ways to create dynamic parameters or add further validation:

- [Powershell functions and Parameter Sets](http://blog.simonw.se/powershell-functions-and-parameter-sets/)
- [Add a Parameter to Multiple Parameter Sets in PowerShell](http://www.jonathanmedd.net/2013/01/add-a-parameter-to-multiple-parameter-sets-in-powershell.html)
- [How To Implement Dynamic Parameters in Your PowerShell Functions](https://mcpmag.com/articles/2016/10/06/implement-dynamic-parameters.aspx?m=1)