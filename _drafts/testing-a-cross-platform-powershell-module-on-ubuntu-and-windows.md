---
title: Testing a PowerShell Core module on Ubuntu and Windows with AppVeyor
image: "/content/images/2018/08/ubuntu_windows_7_style-widescreen_wallpapers.jpg"
---
This blog post details how you can configure the Continuous Integration tool AppVeyor to test one of your PowerShell projects on both a Windows and Ubuntu VM each time you make a commit. This is obviously particularly useful if you're authoring a cross-platform PowerShell Core module and want to be certain that the module functions successfully across multiple platforms each time you make a commit.

I recently authored a module called HashCopy, which I talked about in my previous blog post. As part of authoring the module, I realised that there was very little in the code that would stop it from working under PowerShell Core and on any of the supported operating systems. My module primarily interacts with the file system, so the only thing I had to do was make sure that the module handled file paths in a generic way. Doing this meant using the `Resolve-Path`, `Split-Path` and `Join-Path` cmdlets as part of handling file paths so that responsibility for constructing those paths correctly was left to PowerShell (vs me hard-coding in a backslash into a path that would be invalid on a non-Windows system, for example).

Once I'd made those changes I obviously wanted to verify that the module would work correctly on a non-Windows system. I (ideally) didn't want to have to spin up my own non-Windows VM for testing and I recalled seeing a while back that AppVeyor was going to start providing [Linux builds](https://www.appveyor.com/docs/getting-started-with-appveyor-for-linux/).

> If you're not currently using AppVeyor as a Continuous Integration (CI) tool for your PowerShell modules, have a look at one of these guides for how you can start to leverage this functionality:
>
> - ..
> - ..
>
> Setting up CI is of great value in combination with ensuring your PowerShell modules/code are paired with a detailed set of Pester tests, as you can then have the CI run those tests automatically each time a new commit is made, informing you as soon as possible if you've introduced any new functionality that has caused issues.




