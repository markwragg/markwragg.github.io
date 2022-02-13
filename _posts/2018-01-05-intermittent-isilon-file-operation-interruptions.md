---
title: Intermittent Isilon write failures due to SMB3 Multichannel setting
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2018/01/dell-emc-isilon.png"
  teaser: "/content/images/2018/01/dell-emc-isilon.png"
date: '2018-01-05 22:08:15'
tags:
- isilon
- troubleshooting
- windows
---
I recently resolved an issue with our Isilon storage cluster that was causing file writes to be interrupted and fail. While diagnosing the issue I discovered that (intriguingly) the disruption was occurring at (almost) exact 10 minute intervals. Any in progress write operation occurring at that interval would fail.

*TL;DR: In case you want to skip straight to the resolution, the cause of my issue turned out to be the `support-multichannel` setting on the Isilon, which when enabled caused the issue to occur.*

*I resolved the problem by logging on to the Isilon via SSH and entering the following command to verify the setting was enabled:*

```
isi smb settings global view
```

*And then entered this command to set `support-multichannel` to be off:*

```
isi smb settings global modify --support-multichannel=no
```
*Immediately after which the disruptions no longer occurred.*

---I wanted to Blog this issue for two reasons:

1. When googling, I found nothing like this on the internet, at least related specifically to an Isilon. It took me several days of solid work to resolve, so I hope to save someone else at least some of that time.
2. In a job interview recently I was asked to describe a complex problem I solved and how I went about solving it. I completely blanked on the question (which is annoying because as a former manager I used to ask this classic question all the time so really should have expected it). It occurred to me that what I describe here would (perhaps) now be a good example to use in the future. However I found even just a few weeks later I was already struggling to recall it. This blog post will hopefully rectify that.

### Symptoms

The issue was reported to me by our database team, who were finding that their SQL database backups were sometimes failing at random on a Windows 2012 R2 SQL server. Those backups were being written to a 5 node Isilon cluster. The issue was apparently particularly likely to occur with large (50GB+) databases, but could also occur for a database of any size.

### Diagnosis

The first step I took to diagnose the issue was to perform a few backups of one of the large databases to the local server storage to see if they failed. They did not, which seemed to rule out anything being wrong with the SQL side of things.

The SQL server was a VMWare VM, so I performed a vMotion of the server to another host. The issue still occurred, which seemed to rule out anything with the underlying host hardware.

Next I found some suggestions online that the network card setting "Allow the computer to turn off this device to save power" could cause intermittent interruptions. This setting was enabled on our NIC so I disabled it. This made no difference to the issue.

Following this I wanted to see how frequently the failures/interruptions were occurring, so I used `fsutil file createnew 100mb.txt 104857600` to create a 100mb test file and then wrote a very dirty PowerShell script to copy this file repeatedly to the Isilon and log any errors that occurred along with a timestamp:

```
$file = '100mb.txt'
 
While ($True) {
    Try{
        Copy-Item $file "\\myisilonserver\somefolder\"
        Remove-Item "\\myisilonserver\somefolder\$file"
    }Catch{
        Get-Date -f s
        Write-Error $_
    }
}
```
Running this revealed the very intriguing fact that the interruptions were occurring at almost 10 minute intervals. I say almost as it was 10 minutes and perhaps a fraction of a second, so it would sort of slowly drift. Here's some example timestamps I logged:

```
2017-12-15T08:27:42
2017-12-15T08:37:42
2017-12-15T08:47:43
```

After discovering this I ran a constant ping to www.google.com to see if that was interrupted at the same time as the copies were. It was not, suggesting the NIC was not the cause. I was able to get permission to reboot the Isilon, but this made no difference to the issue. 

I also rebooted the SQL server VM, and this didn't resolve the issue, but I discovered that it changed the time it was occurring. It was still at 10 minute intervals, but it seemed to correlate as being 10 minutes from when the server had completed booting. A while later the SQL team informed me of another server where the issue was occurring. I ran the PowerShell copy test on that server and found that it also was having 10 minute interval interruptions starting from the time that the server booted. 

I also tried the copy test from some other servers. I found a VM running Windows 7 was not impacted by the issue, my copy test was never interrupted. This gave me a hunch that it related to SMBv3, as this was introduced in Windows 8 and Server 2012. I did more testing across more servers that backed this up. It was only Win 8 / 2012 or newer OS systems that were affected by the issue. This discovery was frustrating, because it seemed to suggest the Isilon was not at fault.

After this my frustration sort of peaked, and I went down several rabbit holes where I tried changing the NIC type of the VM, disabling SMBv3, disabling SMBv1/2, changing jumbo frames settings and performing network packet captures and analysis with Microsoft Message Analyzer. None of my changes helped and the packet capture showed the issue occurring but did not help me to understand why.

### Resolution
I'd like this story to end with me using some incredible feat of technical prowess to solve this issue, but actually I got lucky. I was in the fortunate position of having a second (identical model) Isilon cluster in which the issue did not occur. This had me convinced there was something specific to the Isilon causing the issue, so I painstakingly compared every setting between the two clusters by logging in to their Web UIs.

What I discovered was that **they were identical.**

![](/content/images/2018/01/dafuq.jpg)

Eventually (after much screaming in to the void) I logged on to each Isilon via SSH and used the Isilon CLI to compare their settings. In particular I ran:

```
isi smb settings global view
```
which (among other settings) returned `Support Multichannel: Yes` on the problem server and `Support Multichannel: No` on the problem free server.

A short Google later, I discovered I could change this setting by running:
```
isi smb settings global modify --support-multichannel=no
```
After which my copy test showed no further interruptions!

### Root Cause

I'm not actually completely sure why having this setting enabled caused my problem. However I did do some research to understand the feature and I did find a potential explanation for why it was occurring at exact 10 minute intervals.

Multichannel is an SMB feature that can be used to increase network performance and fault tolerance. There's more info about it here: 

- https://blogs.technet.microsoft.com/josebda/2012/06/28/the-basics-of-smb-multichannel-a-feature-of-windows-server-2012-and-smb-3-0/

But to summarize:

> Windows Server 2012 includes a new feature called SMB Multichannel, part of the SMB 3.0 protocol, which increases the network performance and availability for File Servers.

> SMB Multichannel allows file servers to use multiple network connections simultaneously and provides the following capabilities:

> - Increased throughput. The file server can simultaneously transmit more data using multiple connections for high speed network adapters or multiple network adapters.
> - Network Fault Tolerance. When using multiple network connections at the same time, the clients can continue to work uninterrupted despite the loss of a network connection.
> - Automatic Configuration: SMB Multichannel automatically discovers the existence of multiple available network paths and dynamically adds connections as required.

This article goes on to say that the requirements are that the server has multiple NICs, NIC Teaming or a NIC that supports RSS or RDMA. It seems [there might be an issue with RSS for the VMWare vmxnet3 driver at the moment](https://communities.vmware.com/thread/545782), so maybe that was a contributing factor / the root cause.

I found another Microsoft Blog post from 2012 that I think explains the 10 minute interval:

- https://blogs.technet.microsoft.com/josebda/2012/10/08/windows-server-2012-file-servers-and-smb-3-0-simpler-and-easier-by-design/

> Beyond surviving the failure of an interface, SMB Multichannel can also re-establish the connections when a NIC “comes back”. Again, this happens automatically and typically within seconds, due to the fact that the SMB client will listen to the networking activity of the machine, being notified whenever a new network interface arrives on the system.

> Using the example on the item above, if the cable for the second 10GbE interface is plugged back in, the SMB client will get a notification and re-evaluate its policy. This will lead to the second set of channel being brought back and the throughput going up to 20Gbps again, automatically.

> If a new interface arrives on the server side, the behavior is slightly different. If you lost one of the server NICs and it comes back, the server will immediately accept connections on the new interface. However, clients with existing connections might take up to 10 minutes the readjust their policies. This is because **they will poll the server for configuration adjustments in 10-minute intervals.** After 10 minutes, all clients will be back to full throughput automatically.

This also explains why it correlated with the server startup as (I assume) the interval starts from the point SMBv3 is initialised.

---
If you've suffered with this issue or anything similar and/or know any more detail about the root cause, please let me know in the comments below.