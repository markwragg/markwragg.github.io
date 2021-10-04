---
title: Using AppVeyor to build, test and deploy a PowerShell project
image: "/content/images/2017/01/Continuous-Narrow.jpg"
---
I have wanted to explore the topic of Continuous Integration (and particularly how it might apply to PowerShell) since reading the excellent [Release Pipeline Model Whitepaper](http://download.microsoft.com/download/C/4/A/C4A14099-FEA4-4CB3-8A8F-A0C2BE5A1219/The%20Release%20Pipeline%20Model.pdf) by Michael Greene and Steven Murawski.

![](/content/images/2017/01/release-pipeline-build.png)

I am relatively comfortable with Source Control (and it's many benefits) and I've played a fair bit with Pester so I have a framework for testing. However I've struggled to understand how the "Build" or "Release" phases might apply to my (typically) administrative scripts. What eventually encouraged me to dig deeper was a [friend](http://sammart.in) suggesting I put my latest project in to the PowerShell Gallery and it therefore seemed sensible to have an automated process that would ensure committing to Github would also trigger a publish to the Gallery and (more importantly) that changes to both still resulted in clean, functional code.

*--Confession time: I have published to the PowerShell Gallery once before with my AD Audit project. Since doing that I've almost certainly made changes to the project in Github and I recently accepted a (much appreciated) pull request, neither of which I remembered to push in to the Gallery. Additionally, I definitely didn't perform any testing of the code after the latter.*

## AppVeyor

I recall looking at AppVeyor briefly before and dismissing it because I didn't think there was a free option as well as I was typically working on scripts that I couldn't easily open source.

There is a free option f..

## Todo List

- I~~mplement PSScriptAnalyzer in the pipeline.~~
- Flesh out Pester tests (unit + integration - separate files?)
- ~~Implement deployment in to PSGallery~~
  - ~~Needs to factor in incrementing build #. Take that # from the build number used in AppVeyor? Be good if they match.~~
- ~~Implement Badge on readme.md~~
- Blog about how the green tick next to the commits was already helpful :)
- Look in to using PSake -- alternative to appveyor.yml as a build script?
- Merge together the 4 appveyor ps files? Need to then update the yml file.
- ~~make it so it only publishes the module if it's changed. Work out best way to do this? Can we get some sort of hash?~~

## Links
- http://stackoverflow.com/questions/41184979/how-to-publish-a-powershell-module-to-powershellgallery-with-appveyor
- http://ramblingcookiemonster.github.io/GitHub-Pester-AppVeyor/#appveyor
- http://ramblingcookiemonster.github.io/Github-Pester-AppVeyor-Part-2/

https://4sysops.com/archives/unit-tests-versus-integration-tests-in-pester/

https://blogs.msdn.microsoft.com/jtarquino/2016/10/15/powershell-module-with-continuous-integration-static-analysis-and-automatic-publish-to-gallery/

https://www.appveyor.com/docs/appveyor-yml/

https://ci.appveyor.com/project/markwragg/powershell-xkcd

http://javydekoning.com/getting-started-with-appveyor/