---
title: Automate yourself a better job with Powershell
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2016/05/Robot-Hands-Keyboard.jpg"
  teaser: "/content/images/2016/05/Robot-Hands-Keyboard.jpg"
date: '2016-05-10 00:00:06'
tags:
- powershell
- automation
---
The phrase "automate yourself out of a job" is too easily misinterpreted as a negative. Without any context it's possible to view it as to mean *you shouldn't automate things. If you do you'll end up unemployed or could make others unemployed.* In IT at least, this is unlikely to be true. Powershell is 10 years old later this year and if you're not using it, here's why you should.

I've never met anyone that has told me they scripted themselves redundant. Instead automation should free you from mundane tasks and give you time back to focus on other improvement activities and projects. But many people still prefer to perform tasks manually and (in the Windows space at least) I've encountered a lot of people that do not script on a regular basis as part of their role and are getting along just fine (for now) with little to no Powershell ability.

## Fear of automation

I doubt most people genuinely believe automating things would lead to their job loss. In fact its more than likely to achieve the opposite. So what are some of the reasons people are afraid to automate?

![Skynet](/content/images/2016/05/terminator.jpg)

*-- Other than inadvertandly inventing Skynet of course.*

- **It's too hard.** If you're new to scripting, or to a particular scripting language, the barrier to entry can seem quite high. There is a steep learning curve that is offputting, particularly where the manual steps of the task are well known and are quicker to achieve when performing the task in isolation. 
- **I could break more things faster.** *"If i'd scripted this, i'd have broken everything!"*, it's certainly much easier to have a wider impact when using automation scripts and if an error is introduced that causes a problem, that problem could be many times worse than if the task was being done manually. 
- **Permanent ownership.** If the environment you are in has little to no automation, you could worry that you would find yourself solely responsible for the ongoing maintenance of your script, as well as effectively adopting the automated task as an individual. If the script goes wrong, you will always be called upon to investigate why.

## Conquering these fears

If you find any of the above relatable, consider these tips:

- **Write small scripts often.** Use code whenever possible. Powershell is an administration language as well as a scripting language. This means it's just as valid to fire up the console and write one of two commands as it is to write a multi-line script. Need to restart a service? Stop opening that mmc. Get-service servicename | restart-service.
- **Dedicate time to focus.** When you are ready to do some longer script development, shut down your email. Close your chat client. Block out your calendar. Developers know you need a few solid hours uninterrupted to make real progress.
- **Build a network of resources.** The internet is teeming with resources to help you and it's ever growing by it's very nature. That doesn't have to be limited to websites, there are podcasts, meet ups, communities, conferences, hackathons as well as (gulp) books. See my [Resources](http://wragg.io/Resources/) page for a starting point and consider building your own. Also, check out the [Microsoft Virtual Academy](https://mva.microsoft.com/en-us/training-courses/getting-started-with-powershell-30-jump-start-8276).
- **Test often and use safety switches.** Powershell has some great built in options to keep you from making mistakes. Use [-whatif and -confirm switches](http://www.computerperformance.co.uk/powershell/powershell_whatif_confirm.htm) to get a console output of what a command will do before it does it, or a console prompt. Use these often in your initial test. Write [Pester](http://www.powershellmagazine.com/2014/03/12/get-started-with-pester-powershell-unit-testing-framework/) scripts. Since PowerShell version 4 [Pester](http://www.powershellmagazine.com/2014/03/12/get-started-with-pester-powershell-unit-testing-framework/) has existed as a framework to allow you to code validation tests for your scripts. Consider writing your tests first, then code. This is called [Test Driven Development](http://www.hurryupandwait.io/blog/why-tdd-for-powershell-or-why-pester-or-why-unit-test-scripting-language). If you're using Source Control, check your code in and then have it peer reviewed. Even if you're not, get a second person to review and critique your code. If you're doing complex commands (particularly when using the pipeline) consider temporarily breaking your code up and run parts in isolation to check it is returning the results you expect. Finally, if you can, always deploy your code first in a test environment.
- **Inspire and train others.** To ensure you don't end up a Single Point of Failure, you need to ensure others are as invested in automation as you are. Share your resources with them. Help them to develop scripting skills. Run some training sessions in-house, or seek external training. Getting people over that initial hump is the hardest part but once they are invested, enthusiasm multiplies.

## Benefits of automation

![](/content/images/2016/05/grass-1.jpg)

- **Job satisfaction**. Repetitive tasks are boring. It might feel like job security now but it's not guaranteed to be in the future. Scripting is fun and seeing work complete itself is deeply satisfying.
- **Consistent delivery**. Failure is not an option. Automation can ensure a consistent result every time. No more risk of human error introducing a small unnoticed problem, or resulting in hours of troubleshooting. And if you develop your scripts in a small and often way (Continuous Delivery) and use Source Control rolling back from a problem is easy due to version control.
- **Free your time to do more interesting things**. Manual tasks suck time. Free time is precious and gives rise to great opportunity, such as automating something else :).

There will always be more things to do in IT than there is time to complete. Automation gives us more opportunity to make smart choices about what those things are.