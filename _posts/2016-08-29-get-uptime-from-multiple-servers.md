---
title: Get uptime from multiple servers with Powershell
header:
  overlay_image: "/content/images/2016/05/flip-clock.jpg"
date: '2016-08-29 10:00:11'
tags:
- powershell
---
The following script can be used to get the current uptime from a collection of servers in Active Directory using WMI. I used it as a way to audit our estate, keen to understand how long servers have been operational for, in part to identify those which were potentially not routinely receiving Windows Updates.

![](/content/images/2016/08/20130306-005558.jpg)

> Note that as this queries WMI remotely you'll need to ensure the ports for WMI are open on any firewalls in between. By default WMI uses port 135 then a randomly assigned high numbered port, but you can make that static per this guidance: https://msdn.microsoft.com/en-us/library/bb219447(v=vs.85).aspx

**The script**

The crux of the script is the `Get-WmiObject -ComputerName $computer.name win32_operatingsystem` part which we then use to get the `lastbootuptime` parameter, but there are some optional constructs which I have detailed after the script below.

<script src="https://gist.github.com/markwragg/e873167fccd09656bf36d848f7995bd0.js"></script>

The script uses `write-progress` to show progress. In order to ensure any verbose or warning messages do not appear underneath the progress bar, I use `write-host` to output 5 blank lines when the script first runs.

![](/content/images/2016/08/get-uptime2-1.png)

If you run the script with the `-verbose` parameter, you will see each uptime printed to the console as it's obtained (this is not shown above). 

**Script framework**

As with most scripts, I've put a framework around the core functionality to add flexibility, capture errors and show progress. These are explained below.

- Line 4: I have servers organised in to OUs named after their location, so my `$location` parameter allows me to filter the results to one or more of these locations (note you can enter more than one at once, just by comma separating). This isn't a mandatory Parameter so not specifying this just searches all of AD, however you may want to run the script in different locations close to the servers for speed if you can also filter in this way.
- Line 10: I have servers in my estate which are non-Windows but are Domain joined, so I also filter on Operating System name, by default looking for ones with "Windows" in their name (you could remove this if it's not relevant to you).
- Line 19: I use Write-Progress to show which server is currently being polled and how far through the set of servers we are.
- Line 21: I use Test-Connection to check the server is responsive, this speeds up the script for any dead or inaccessible servers.
- Lines 36-42 & 49: I trap any errors that occur in to an object, and at the end of the script spit these out to a separate file for analysis.
- Line 46: I output my results sorted by most to least uptime.
