---
title: 'TILFMOL #4 - PowerShell Jobs'
header:
  image: "/content/images/2017/10/Automation.jpg"
date: '2017-06-19 22:57:21'
tags:
- powershell
---
This is the fourth and final part of a [short series of posts about things I discovered by reading the excellent Learn PowerShell in a Month of Lunches](http://wragg.io/tilfmol-things-i-learnt-from-learn-powershell-in-a-month-of-lunches/) book (recently released in 3rd edition) as a not quite beginner.

This post focuses on things I learnt about PowerShell jobs, which is a feature I'd never used but could see being very useful. PowerShell is by nature, single threaded. However jobs allow you to multitask by pushing one or more commands in to the background thereby allowing them to run sequentially. 

## Local Jobs

There are a couple of different ways to use Jobs. Most simply you can use the `Start-Job` command to execute a script block in the background as follows:

    Start-Job -ScriptBlock { Get-ChildItem C:\ -Recurse }

Explicitly naming the `-ScriptBlock` parameter is optional as it is positional. The output you get from this command will look like this:

![](/content/images/2017/06/PowerShell-Start-job-Example.png)

Which is a listing of the job object that was created. Your job is now running in the background and to check on it's progress you use `Get-Job`. This will list all jobs that exist within the current session and their status (exactly the same as shown above). You can also request a specific job by passing a name or ID.

To retrieve the results of a job you use `Receive-Job <id>`. You usually want to do this when the job is complete (as indicated by `Get-Job`) but there's nothing to stop you doing it while it's in progress and you'd see any output being produced that you would see had you run it interactively.

You can stop a running job with `Stop-Job` and remove them (and their results) with `Remove-Job`. You can also `Suspend-Job` and `Resume-Job`. Finally if you want a session to wait for one or more jobs to complete before doing anything else you can use `Wait-Job`. This might be useful if you had a script that executed a number of distinct tasks simultaneously but you then wanted to wait for them to complete before doing anything else (or before allowing the script to terminate, as the end of the PowerShell session/script will also kill all running jobs whether they finished or not). To do that you might do this: `Get-Job | Wait-Job`.

I can see jobs being useful also as we move into the containerized Windows Nano era of Windows computing. With a command-line only single console version of Windows you might as an administrator want to kick off some tasks without losing the interactivity of your console. Jobs would be your friend here.

## Remote jobs

Of course the greatest benefit is gained by using the jobs functionality to do multiple tasks sequentially across multiple machines. For this you can leverage PowerShell's remoting functionality via `Invoke-Command`. Again, you pass `Invoke-Command` a `-ScriptBlock` to execute and use the `-AsJob` parameter to run the command in parallel on (by default) up to 32 computers (defined via the `-ComputerName` parameter) at once. You can use `-ThrottleLimit` to change this limit (up or down). When the limit is hit, the jobs are queued and as one finishes another is started.

For example:

    Invoke-Command -ScriptBlock {Get-EventLog System -Newest 1} -ComputerName server1,server2 -AsJob

Will get me the latest System event log event from two computers simultaneously. These commands are executed on the remote computers with just the results returned to mine, which is obviously more efficient for anything long running. You'll notice also when you use `Receive-Job` to get the results that a `PSComputerName` property has been automatically added to the object so you can see which results came from where.

Look out for the `-AsJob` parameter switch on other cmdlets (e.g `Get-WMIObject` has it) as a shorter way to invoke jobs without needing `Invoke-Command`. There's also[ this MSDN page on background jobs](https://msdn.microsoft.com/en-us/library/dd878288(v=vs.85).aspx) that indicates how PowerShell toolmakers can add the functionality to their own cmdlets.

## Scheduled Jobs

You can also use built-in PowerShell functionality to create scheduled jobs. This is distinct from the Task Scheduler functionality of Windows (which you can leverage from PowerShell). These are a little complex to build as you need to create a Job Trigger that defines when the task will run. You do this with `New-JobTrigger`. You can also configure complex options for the job via the `New-ScheduledTaskOption` cmdlet. Finally you register your scheduled job with (unsurprisingly) `Register-ScheduledJob`.

Here's an example from the book:

     Register-ScheduledJob -Name DailyProcList `
          -ScriptBlock { Get-Process } `
          -Trigger (New-JobTrigger -Daily -At 2am) `
          -ScheduledJobOption (New-ScheduledJobOption -WakeToRun -RunElevated)

You can see that the JobTrigger and ScheduledJobOption cmdlets are being used in line (in brackets so they execute first) to create the objects necessary for those parameters.

This task will create a job that runs every day at 2am and stores a snapshot of running processes, waking the computer if necessary to do so.

As previously, you see/pick up the result of this job via `Get-Job` and `Receive-Job`. Beware that unlike the other jobs above, Scheduled Jobs are stored on disk (under your profile) so anyone with adequate permissions can see/modify/retrieve them. This of course means that they also differ in that the PowerShell session doesn't need to remain for them to persist.

You can remove a scheduled job via `Unregister-ScheduledJob`. You can also `Disable-ScheduledJob` and `Enable-ScheduledJob` them.

This has been a very brief tour of PowerShell jobs that I hope you found a useful introduction. To learn more, pick up Month of Lunches or explore all the job commands via `Get-Command *job` and their `Get-Help` pages.