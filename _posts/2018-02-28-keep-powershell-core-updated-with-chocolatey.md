---
title: Keep PowerShell Core updated on Windows with Chocolatey
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2018/02/brownie-dessert-cake-sweet-45202.jpeg"
date: '2018-02-28 15:01:22'
tags:
- powershell
- chocolatey
---
PowerShell Core is the cross-platform version of PowerShell that runs on Windows, Mac and Linux. If you are not familar with it, [check out my previous blog post on the topic](http://wragg.io/powershell-core/). It's likely that PowerShell Core will see more regular releases than we've had historically with Windows PowerShell. While you will be able to download the .msi installer for these releases to update your version, this blog post covers how can use the Windows package management tool [Chocolatey](https://chocolatey.org) to manage your upgrades instead.

Since [February 2017](https://blogs.msdn.microsoft.com/powershell/2017/02/01/installing-latest-powershell-core-6-0-release-on-linux-just-got-easier/) its been possible on Linux to install and upgrade PowerShell via the package management tools `apt-get` and `yum`. Windows doesn't natively have a package management tool and Chocolatey exists to fill that void. Package managers greatly simplify the task of installing and managing software packages.

> *"Chocolatey is a package manager for Windows (like apt-get or yum but for Windows). It was designed to be a decentralized framework for quickly installing applications and tools that you need. It is built on the NuGet infrastructure currently using PowerShell as its focus for delivering packages from the distros to your door, err computer."*

If you don't have chocolatey already, there are several installation options detailed here: https://chocolatey.org/install (which I suggest you check for the latest information). For example, you can open an administrative PowerShell prompt and simply run:
```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```
Once this has completed you can check chocolatey is installed by running `choco` at the command-line, which should return something like this:

![](/content/images/2018/02/Check-Chocolatey-Is-Installed.png)

Now that you have Chocolatey, you can use it to install PowerShell Core.

> Note: if you already have a version of PowerShell Core installed manually, you should uninstall it first.

You can see the [PowerShell Core Chocolatey Package here](https://chocolatey.org/packages/powershell-core). Per that page, to install PowerShell Core with Chocolatey run (from an administrative level cmd or powershell window):
```
choco install powershell-core
```
(add `-y` to have it auto accept/bypass any prompts).

This should complete as follows:
![Chocolatey installation of PowerShell Core](/content/images/2018/02/Chocolatey-install-powershell-core.png)

After which you should find PowerShell Core in your start menu:

![PowerShell Core in Start Menu](/content/images/2018/02/PowerShell-Core-Startmenu.png)

If/when PowerShell Core becomes outdated, you can check if a newer version is available by running
```
choco outdated
```
![Checking if PowerShell Core is outdated with Chocolatey](/content/images/2018/02/choco-outdated-powershell-core.png)

and upgrade it by running (in an administrative console):
```
choco upgrade powershell-core
```
If you ever need to install a specific version of PowerShell Core, you can do so with the `--version` switch. For example:
```
choco install powershell-core --version 6.0.0
```
> Note that if you do this from within PowerShell Core the upgrade will complete but your console will close as a result.

# Summary

This has been a quick intro to Chocolatey and how you can use it to manage packages. By using the above workflow you can manage and maintain your PowerShell Core version from the console, in the same way as you can on Linux.