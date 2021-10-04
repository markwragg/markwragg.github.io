---
title: 'Operational Testing for Antivirus: Validating Symantec Endpoint Protection
  with Pester'
header:
  image: "/content/images/2016/10/ThinkstockPhotos-158695294.jpg"
date: '2016-10-19 14:01:00'
tags:
- powershell
- pester
---
This post contains an operational validation test for Symantec Endpoint Protection (SEP) using the [Pester testing framework module](https://github.com/pester/Pester) with Powershell. It performs a few basic checks to ensure SEP is running and healthy. It is intended as a starting point and could be developed further. Your individual requirements will likely vary depending on how you have SEP configured and deployed in your environment.

This test script came as a result of a training session I ran last week to encourage my team to continue to use Pester and Powershell to build on our set of automated operational tests. We don't make changes to SEP very often, but having a way to validate the product after changes as well as to check consistency across our environment seemed useful.

> If you're not familiar with Pester for Operational Testing, have a look at my [getting started with Pester](http://wragg.io/getting-started-with-pester-for-operational-testing/) post.

At the conclusion of the training we each worked on a couple of items from a list of ideas of tests we could perform against our applications and infrastructure. I chose to tackle SEP.

The script has a fairly basic set of tests at the moment and largely validates the configuration by reviewing settings in the registry. At time of writing, the tests are as follows:

- Checks the Symantec Endpoint Protection and Symantec Management Client Windows services are running.
- Checks the smc.exe and ccsvchst.exe processes are running.
- Checks that the SEP version is greater than the master version as defined in the param block at the top of the script (current latest version is 12.1.7).

*-- I was really pleased with how easy it was to do this comparison in Powershell. It might need further testing, but I think as a string 12.1.7.x.x.x is considered "greater than" 12.1.7 so regardless of the installed minor version it was easy to validate it was a certain major version or newer without things needing to get too [complex](http://www.regular-expressions.info/).*

- Checks that a scheduled scan occurred in the last 1 days.
- Checks that the current latest virus definitions are no more than 3 days old.
- Checks that the system is not currently considered infected.
- Checks that the Spyware Protection feature is enabled.
- Checks that the Virus Protection feature is enabled.
- Checks that Firewall Protection feature is enabled.
- Checks that the System Network Access feature is enabled.

*-- Again, your feature set might vary from this, so you could modify the expected results of these tests to either be enabled or disabled per your own expectation.*

Here's the script. To run ensure you have the [Pester](https://github.com/pester/Pester) module installed and then execute with `Invoke-Pester`:

<script src="https://gist.github.com/markwragg/5904856087b73857756e5b5ac0250f5b.js"></script>

Here's how it looks:

![](/content/images/2016/10/SEP-Pester-Tests.png)

If you'd like to develop this further, feel free to [Fork the above Gist](https://gist.github.com/markwragg/5904856087b73857756e5b5ac0250f5b#file-sep-tests-ps1).