---
title: 'TILFMOL #2 - PowerShell Help'
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2017/02/help-keyboard-2.png"
date: '2017-03-02 08:00:00'
tags:
- powershell
---
This is part two of a short series of posts about things I discovered or had clarified by reading the excellent [Learn PowerShell in a Month of Lunches book](https://www.manning.com/books/learn-windows-powershell-in-a-month-of-lunches-third-edition) (recently released in 3rd edition).

This post focuses on things I learnt about the PowerShell help function, including:

- The difference between running `help` and `get-help`
- The meaning of square brackets in help:
  - Optional/Positional Parameters
  - Array Input
- Other punctuation in help
- Extending the output of help

In every beginners guide/book/course that I've seen for PowerShell, the help system comes highly recommended and is usually explained in great detail. PowerShell has been designed to be a very discoverable language (the standardisation of verbs such as "get", "set" etc. being a key example of this).

However in the past I have (as I suspect have a lot of people) actively avoided running `get-help`. I think there a couple of reasons for this:

1. 'Help' in Windows has always felt like something aimed at end users, not at IT pros. As a result there's a stigma to using it.
2. Searching for a topic on the internet on the other hand has always felt like doing research, which is how professionals gain knowledge.
3. The information returned is verbose and contains a lot of unexplained terminology and punctuation (more on this later).

As a result the `help` system is often overlooked, but once you do understand the basic structure and terminology it's a very powerful resource.

# The difference between running help and get-help

One thing that I hadn't realised before reading Month of Lunches is that `Help` is not a straight up Alias for `Get-Help` (it sort of is, but not really). Both commands allow you to access the help system, but when in the PowerShell console `help` paginates by default (e.g if the content returned is longer than the window then it asks you to press a key for --More--):

![](/content/images/2017/02/powershell-help-more.png)

This is equivalent of doing `get-help <command> | more` (but is -intentionally- a lot shorter).

This happens because `Help` is actually a function not an alias (but it has an alias of `Man` to help out those coming from a Unix background) which allows it to work differently (where as Aliases are just different names for the exact same functionality).

There's a small caveat to this: the same is not true in the ISE. Here both help and get-help do not paginate, even if you pipe the content to `More`. 

# Square brackets in help

As mentioned earlier, one of the reasons I think people avoid `Get-Help` in favour of searching the internet (I'm particularly fond of the no-nonsense content on https://ss64.com/ps/ for example) is that when you first run `get-help` for a command you get a SYNTAX block that has a LOT of square brackets and other punctuation that are not explained.

*-- Also, I have (and have seen others) in the past tried to pipe a command to get-help, because the first thing you're told with PowerShell is that it's all about the pipeline. But `my-command | get-help` does not work (which makes sense, even if get-help accepted pipeline input, that would pipe it the results of the command not the command name itself).*

Learn PowerShell in a Month of Lunches clarified for me what all that punctuation means in the help file (i'd come to some conclusions on my own, but having them verified was great). Square brackets are particularly confusing as they have two meanings.

Here's an example:
```
SYNTAX
    Get-Service [[-Name] <String[]>] [-ComputerName <String[]>] [-DependentServices] [-Exclude <String[]>] [-Include
<String[]>] [-RequiredServices] [<CommonParameters>]
```
That's a lot of square brackets (and punctuation in general). 

There are a couple of meanings for the square brackets:

**1\. Around any parameter name or parameter value, they are indicating stuff that is optional.** In the example above, you can see that all of the parameters are optional (each one is surrounded entirely by square brackets). I can execute `get-service` on its own and it just spits out all the services.

You might have noticed that there's additional square brackets around `-Name` that aren't around most of the other parameter names. That's because `-Name` is a positional parameter, so I can provide that input without explicitly naming it, so long as I provide that input in the right position (in this case, first) vs any other positional parameters. E.g I can do:
```
Get-Service spooler
```
Note I can also do this:
```
Get-Service -Exclude sppsvc sp*
```
Which is a little more confusing, but what's happening here is this:
```
Get-Service -Exclude sppsvc -name sp*
```
Because -Exclude was explicitly named, my unnamed input still went to the first positional parameter `-name` like I intended.

If you look again at the earlier snippet of Parameter help text for `Get-Service`, you might be forgiven for thinking that `[-RequiredServices]` is a positional parameter, but it's not. It's a switch parameter, which means it's just true or false (you either provide it or not, with no input) so as a result the whole parameter name is obviously optional.

The other way to know if a parameter is positional is to use `get-help -full`. In the section that provides more detail about each Parameter you will see `Position?` with the value being either `named` or a numerical value representing it's position (starting at 0).

**2\. Next to any type name, square brackets indicate that the parameter accepts multiple/array input of that type.**. As demonstrated earlier, the Help for `Get-Service` shows that -ComputerName accepts string array input (per the inner square brackets below):

```
SYNTAX
    Get-Service [-ComputerName <String[]>]
```
So I can (for example) do this:
```
Get-Service -ComputerName 'Server1','Server2','Server3'
```
And the cmdlet will run multiple times internally for each input I provided.

These concepts weren't new to me, but this clarified what the help file was trying to tell me with all those square brackets (and whether my assumptions about them were correct). The result is knowing (through `get-help`, and with less trial and error) how to use cmdlets more efficiently.

# Other punctuation in help

In case you're not aware, here's some explanations for the other punctuation you see via help:

- Dashes '-' precede parameter names, just as they do when you execute the command via the command-line (this is the PowerShell version of a switch proceeded with a forward slash '/' in the old style DOS commands.
  - Be aware you can still use DOS commands in PowerShell and when doing so you should revert to their original syntax such as / switches, as well as be aware that their output will not be a PowerShell object (the exception to this rule is where the old DOS command is now a PowerShell alias: e.g `dir` is an alias for Get-ChildItem, and so it is a true PowerShell command).
- Angle brackets <> surround the type of input a parameter expects. E.g `<string>` expects a string of text input (and as mentioned earlier `<string[]>` means that the input accepts one or more strings: either a single string or an array).

# Extending the output of help

Finally, there's a few good ways to extend the output of help:

- Use the `-detailed` parameter with `get-help` to see extra information about parameters, such as whether they accept pipeline input and whether that is by value, name or both (per my [previous article on the pipeline](http://wragg.io/tilfmol1-the-powershell-pipeline/)) and usage examples for the command. 
- Use the `-full` parameter to see all the help information about a command at once (this is pretty verbose).
- You can also do `-parameter <parameter name>` to see just the information about a single parameter and `-examples` to just see examples.

Also you can open the help information in a pop up window (a way of keeping the it in your view while returning your console to being functional) via the `get-help` parameter `-showwindow` (there seems to be a bug with this in that it doesn't give you the parameter information that you get with `-detailed` or `-full` at the console even though it seems like it should).

Also if a help file has an online resource you can open that in a web browser by using the `-online` parameter with `get-help`.

---This concludes part two. There's a lot more to the PowerShell help system not covered here, as well as other commands you should be aware of such as `get-command` and (the very important) `get-member` which contribute greatly to the discoverability of PowerShell. These are covered in Month of Lunches, or give them a google.

In part three of this series I cover some [things I learnt about PowerShell Remoting](http://wragg.io/tilfmol-3-powershell-remoting/), which is something that I hadn't had a need to experiment with previously.

If you missed part one of this series, I covered some aspects of [the PowerShell Pipeline](http://wragg.io/tilfmol1-the-powershell-pipeline/).