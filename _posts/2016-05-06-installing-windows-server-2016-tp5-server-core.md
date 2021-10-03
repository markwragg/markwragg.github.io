---
layout: post
title: Installing Windows Server 2016 TP5 Server Core
image: "/content/images/2016/05/you_are_likely_to_be_eaten_by_a_grue_by_nmajmani-d4qbkrg-3.png"
date: '2016-05-06 16:39:57'
tags:
- windows
- server2016
---

Previously I covered [performing the installation of Windows Server 2016 Technical Preview 5](http://wragg.io/installing-windows-2016-technical-preview-5-tp5/) in to a desktop VM with the Desktop Experience (GUI) mode enabled. This post covers the installation of the GUI-less Server Core mode and subsequently how to manage and maintain it.

> Beware that since TP3 they have [removed the ability to turn the Windows GUI on and off after installation](https://technet.microsoft.com/en-us/library/mt427865.aspx) that was possible in previous versions, so like Indiana Jones, [choose wisely](https://www.youtube.com/watch?v=0H3rdfI28s0).

**1. Install Windows**

To get Windows installed simply [follow the first 4 steps of my previous article](http://wragg.io/installing-windows-2016-technical-preview-5-tp5/), ensuring that on this occasion you choose to install a version of Windows *without* Desktop Experience. 

That should get you here:

![](/content/images/2016/05/2016-password.png)

And after setting a password and signing in, here:

![](/content/images/2016/05/2016-core.png)

*-- It is pitch black. You are likely to be eaten by a grue.*

**2. Install Updates**

You should first ensure you install the [Cumulative Update for TP5](**http://**). The release notes state its important to do this before installing any Roles or Features or bugs will occur (and you can't just install it later).  There's a handy tool to do server configuration and updates from the command-line. 

Simply enter `sconfig` to access this interface:

![](/content/images/2016/05/sconfig.png)

Select option 6 and then A to search for all updates.

![](/content/images/2016/05/Updates-1.png)

Install the updates and reboot.

If (like me) you're using VMWare Workstation or similar now is a good time to install VMWare tools (or similar). With VMWare tools you mount the ISO to the CD drive via the client as usual (Player > Manage > Install VMWare tools) then enter:
```
cd D:
setup64.exe /S /v "/qn REBOOT=Y"
```
The server will reboot automatically.

**3. Install Remote Management tools**

The easiest way to manage your server from this point forward is to use the Remote Tools. These have been a Windows Feature that you can install for some time, but it seems formal support for managing Windows Server 2016 is currently only provided by the [RSAT tools for Windows 10, which you can download here](https://www.microsoft.com/en-gb/download/details.aspx?id=45520).

**4. Install Roles/Features**

You can install roles and features with the Server Manager remote tool (in the same way you can locally when you have a GUI), but if you'd like to do it more directly from the server you can use Powershell. Bear in mind by default you're in a cmd shell, so to access powershell simply run `powershell`. Your prompt will change to be prefixed with PS.

To see the features currently installed execute `get-windowsfeature`. To install a feature enter `install-windowsfeature <name>`. Remove a feature with `remove-windowsfeature <name>`.

![](/content/images/2016/05/powershell-windowsfeature.png)

## Further Reading

Bruce Adamczak has published a very thorough [Server Core Survival Guide](https://blogs.technet.microsoft.com/bruce_adamczak/2013/01/15/2012-core-survival-guide/) for Windows 2012 which I suspect is (for the most part) still applicable to 2016.