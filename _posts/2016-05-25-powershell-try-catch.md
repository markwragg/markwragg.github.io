---
title: Handling Powershell exceptions with Try..Catch..Finally
header:
  image: "/content/images/2016/05/yoda-do_or_do_not.jpg"
date: '2016-05-25 21:58:44'
tags:
- powershell
---
Recently while writing a script I expected two commands I was calling to throw exceptions because I expected my script to be unable to remotely connect to some of my servers. Initially I handled the result of these exceptions with `If..Else` blocks, but it felt like a `Try..Catch` might be more appropriate.

*-- Spoiler alert: On this occasion it actually turned out it wasn't, but it's a useful technique regardless.*

![](/content/images/2016/05/yoda-wisdom-1.gif)

While Master Yoda does not believe in a "Try", Powershell fortunately does. But there are some caveats to using this technique which I will detail below. 

In case you're not aware performing a `Try..Catch` is as simple as:

```
Try{
    Do-something
}Catch{
    Do-something else, but only if a terminating error has occurred.
}
```
Additionally you can extend this to include:
```
}finally{
    Do-something always, whether a terminating error occurred or not.
}
```

A Finally block can be useful as it will always execute when an error does not occur, but it will also execute if CTRL+C or an Exit keyword is used to stop the script from within a `Catch` block.

Here are some other useful things to know about `Try..Catch`:

**#1: The Catch block will only execute if a terminating error has occurred**

Powershell errors come in two forms, terminating and non-terminating:

>**An error is a terminating error if:**

>- it prevents your cmdlet from continuing to process the current object or from successfully processing any further input objects, regardless of their content.
>- you do not want your cmdlet to continue processing the current object or any further input objects, regardless of their content.
>- it occurs in a cmdlet that does not accept or return an object or if it occurs in a cmdlet that accepts or returns only one object.

>**An error is a non-terminating error if:**

>- you want your cmdlet to continue processing the current object and any further input objects.
>- it is related to a specific input object or subset of input objects.
>
>*Source: https://msdn.microsoft.com/en-us/library/ms714414(v=vs.85).aspx*

If your catch block is not executing, your error is either non-terminating or not throwing an error at all. But..

**#2: You can force a cmdlet to throw a terminating error by using the -erroraction parameter**

The `-erroraction` parameter is available for any cmdlet that supports [common parameters](https://technet.microsoft.com/en-us/library/hh847884.aspx). There are the following options for this parameter:

- 0 : SilentlyContinue. This suppresses the error message and continues execution.
- 1 : Stop. This displays the error message and stops executing the specific command.
- 2 : Continue. Displays the error message and continues executing the command. "Continue" is the default value.
- 3 : Inquire. Displays the error message and prompts you for confirmation before continuing. This is probably only useful when debugging.
- 4 : Ignore. Suppresses the error message and continues executing the command. Unlike SilentlyContinue, Ignore does not add the error message to the $Error automatic variable. 

*-- Tip: You can use the numbers above as a shortcut to these states. Additionally an alias of `-erroraction` is `-ea`. So for example, if you want a cmdlet to silently continue when an error occurs, you can add the parameter `-ea 0`. Beware that this potentially makes your code a little less explicit to others.*

Note that when you use this parameter on a cmdlet it only applies to that specific command. As noted above, the default is "Continue", but you can override this by setting the $ErrorActionPreference variable. 

You can set $ErrorActionPreference multiple times, so for example you could change the state to -SilentlyContinue for a block of code, then change it back.

**#3: When a terminating error occurs, nothing else in the Try block executes**

This is one way to use a `Try..Catch`, as you can create a `Try` block of code that you only want to fully execute when no error occurs. For example, you could use the `test-connection` cmdlet to check if a server pings and if it fails skip anything else afterwards that relied on that connectivity. Without the `Try..Catch` the specific cmdlet would throw an error and then all other subsequent lines would try to execute.

It's worth noting that it interrupts the pipeline, which means if you're piping multiple inputs to a single cmdlet, no further input is sent to the cmdlet as soon as one of the inputs causes a terminating error to occur.

**#4: Any code after the Try..Catch will execute**

It's worth stating, because you might expect a **terminating** error to stop the script entirely, but as stated above, it only applies to the command. This means that if you put any lines of code after (and outside) of your `try..catch` it will execute.

**#5: You can specify multiple catch blocks to handle different exceptions**

You're not limited to a single Catch block following a Try, you can specific multiple blocks for different exceptions. For example:

```
Try{
    Pizza
}Catch[Brocolli.topping.eww]
    Throw-Bin
}Catch[Pineapple.topping.eww]
    Give-Steve
}Catch{
    Any-other-pizza-error
}
```

The last generic catch block handles any other error not specified. It can be tricky to work out from the default error message what Exception name to use, but [Boe Prox has written a great article on how to get the Exception type for an error that has occurred so that you can build it in to a Catch block.](https://learn-powershell.net/2015/04/09/quick-hits-finding-exception-types-with-powershell/)

## So.. why/why not do this?

Suppressing error messages is generally considered an anti-pattern. By doing so during development you're making debugging harder. After development, you might be misleading the user on the success of the script. The default "Continue" behaviour ensures error messages are presented to the user. "Stop" allows you to utilise a Catch block for a non-terminating error to do something more than just report the error to the user (such as log to a file, send an email notification, or initiate a clean up process). 

Using "SilentlyContinue" is dangerous (particularly when used as the default preference), but equally an error that you *expect* to get could mislead a user to think a script has failed when it has not. Here's the issue, if you want to suppress the default error output from the user, you can't use a `Try..Catch` because as far as Powershell is concerned no error has occurred.

For the use case that had me explore this topic, I ultimately reverted to an `If..Else` construct, and used `-ErrorAction "SilentlyContinue"` for the specific cmdlets that I expected to error. In my `Else` block I used Write-Warning to let the user know what failure had occurred, because I wanted a cleaner overall result than the default error output. If the script had any other issues, those errors would still be displayed. 