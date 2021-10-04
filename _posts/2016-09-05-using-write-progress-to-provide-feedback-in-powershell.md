---
title: Using Write-Progress to provide feedback in Powershell
header:
  image: "/content/images/2016/09/time-passing.jpg"
date: '2016-09-05 19:48:07'
tags:
- powershell
---
I like my scripts to give feedback to the console to demonstrate progress where possible and Powershell provides a number of cmdlets to do this, one of which is `Write-Progress`.

> The Write-Progress cmdlet displays a progress bar in a Windows PowerShell command window that depicts the status of a running command or script. You can select the indicators that the bar reflects and the text that appears above and below the progress bar.
>
> *Source: https://technet.microsoft.com/en-us/library/hh849902.aspx*

The progress bar and status messages displayed by `Write-Progress` float automatically at the top of the page, overlaying whatever text is present. There are several different ways to use `Write-Progress` some of which may surprise you:

## 1. You don't *actually* need to know how much progress you've made to use Write-Progress

Surprisingly, you can use `Write-Progress` without actually displaying a progress bar. This can be useful if you want to utilise a floating progress notification but have no way of knowing how much progress has been made. For example:

```language-powershell
Write-Progress -Activity "I'm going to sleep for 5 seconds" -Status "Zzzzz"
Start-Sleep 5
```
![](/content/images/2016/09/write-progress-no-progress.png)

To be explicit about not knowing how much progress to report, you can specify `-PercentComplete -1`.

## 2. You can tell an instance of Write-Progress to disappear

If you want a Progress message to disappear while your script continues to do other processing, you can dismiss them using the `-Completed` switch. For example:

```language-powershell
Write-Progress -Activity "I'm going to sleep for 5 seconds" -Status "Zzzzz"
Start-Sleep 5
Write-Progress -Activity "Sleep" -Completed
Start-Sleep 2
```

## 3. You can use Write-Progress to count down a number of seconds remaining

This is most useful when your script needs to wait for a predetermined amount of time before proceeding and you want to show the user how long they have left to wait. To do this use the `-SecondsRemaining` switch. For example:

```language-powershell
For ($i=5; $i -gt 1; $i–-) {
    Write-Progress -Activity "Launching rocket" -SecondsRemaining $i
    Start-Sleep 1
}
```
![](/content/images/2016/09/write-progress-seconds-countdown.png)

![](/content/images/2016/09/rocket-launch-1.jpg)


## 4. To show a progress bar, just calculate the percentage of work completed

To show a progress bar you use the `-PercentComplete` switch to report a value between 0 and 100. Within a `For` loop you can likely do this using the variables that count the iterations. For a `ForEach-Object` loop, its best to use a counter variable which you increment on each iteration and then the `.Count` property of your collection for the total. For example:

```language-powershell
$WinSxS = Get-ChildItem C:\Windows\WinSxS
$i = 1

$WinSxS | ForEach-Object {
    Write-Progress -Activity "Counting WinSxS file $($_.name)" -Status "File $i of $($WinSxS.Count)" -PercentComplete (($i / $WinSxS.Count) * 100)  
    $i++
}
```
![](/content/images/2016/09/write-progress-bar.png)
## 5. You can show more than one progress bar at a time

Referred to as nested progress bars, you simply need to use the `-id` parameter to ensure they are not layered on top of each other. The first progress bar does not need an ID, but you then need to number all subsequent ones sequentially in the order you want them layered.

The following script demonstrates four nested progress bars by displaying a running clock of the current time.

<script src="https://gist.github.com/markwragg/73addf16504caaf72da1633cdac57e68.js"></script>

![](/content/images/2016/09/watch-timepassing.png)


The script ends automatically at midnight (to have it running perpetually, change the last `Until` statement to `While ($true)`).