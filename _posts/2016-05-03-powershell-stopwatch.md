---
title: A PowerShell stopwatch for your Profile.ps1
image: "/content/images/2016/05/stopwatch-3.png"
date: '2016-05-03 20:14:13'
tags:
- powershell
---
[Boe Prox](https://twitter.com/proxb) blogged/tweeted recently about [how you can start a stopwatch in PowerShell with one line](https://learn-powershell.net/2016/04/29/quick-hits-create-and-start-a-stopwatch-in-one-line/) using the command `[System.Diagnostics.Stopwatch]::StartNew()`. While there are plenty of other ways to measure time in PowerShell (e.g measure-object, new-timespan), I could see an adhoc stopwatch being useful on occasion. However the raw class name is a bit clunky and forgettable. Therefore as a bit of fun I have created it as a series of functions in my PowerShell Profile.ps1 with friendly names I could call when needed.

-- *The irony of the fact that I've turned his one-liner in to approx. 30 lines of code is not lost on me btw.*

---
- In case you're not aware, the PowerShell profile is a place to store functions or commands that you want executed or available automatically each time you start PowerShell. There are actually [several different kinds of Profile](https://blogs.technet.microsoft.com/heyscriptingguy/2012/05/21/understanding-the-six-powershell-profiles/).
- Another example of something I put in my PowerShell Profile is set-location (essentially a "cd") to my Scripts directory, as its where I tend to want to start as my working directory.

---
## Stopwatch functions

In my profile.ps1 I've created the following functions:
```language-powershell
function start-stopwatch {
    $script:StopWatch = [System.Diagnostics.Stopwatch]::StartNew()
    write-output "Stopwatch started at $(get-date). Use stop-stopwatch or esw to stop."
}
```
This starts the stopwatch running and writes to the console a message with the date it started. Note the use of $script scope for my Stopwatch variable, to make it accessible to the other functions. There may be a better/more responsible way to do this.

Now it's off and running, we obviously need a way to stop it:

```language-powershell
function stop-stopwatch {
    $script:StopWatch.Stop()
    write-output "Stopwatch stopped at $(get-date). Elapsed time is $($script:StopWatch.elapsed.tostring())"
    write-output $script:Stopwatch.Elapsed
}
```
This function allows us to continue the stopwatch without resetting its current time value:
```language-powershell
function continue-stopwatch {
    $script:StopWatch.Start()
    write-output "Stopwatch continuing from $($script:StopWatch.elapsed.tostring())"
}
```
And this function allows you to get the current value from the stopwatch without interrupting it:
```language-powershell
function get-stopwatch {
    If ($script:Stopwatch.IsRunning) {$state = "running"}Else{$state = "stopped"}
    write-output "Stopwatch is currently $state. Elapsed time is $($script:StopWatch.elapsed.tostring())"
    write-output $script:Stopwatch.Elapsed
}
```
Although these function names are easier to remember/recall, I still found myself wanting to be able to control it more quickly. As such the last part of my PowerShell Profile creates some short aliases for these new functions:
```language-powershell
new-alias ssw start-stopwatch
new-alias esw stop-stopwatch
new-alias csw continue-stopwatch
new-alias gsw get-stopwatch
```
As an aside I used get-alias to check that these 3 letter aliases were not in use for anything else, which they don't seem to be currently but beware they could be in the future.

Here's what the output looks like:

![](/content/images/2016/05/stopwatch-1.png)