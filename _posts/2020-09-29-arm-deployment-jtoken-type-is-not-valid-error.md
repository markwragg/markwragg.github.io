---
title: ARM Deployment "JToken type is not valid" error
teaser: "/content/images/2020/09/windows-keyboard-black.jpg"
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2020/09/windows-keyboard-black.jpg"
date: '2020-09-29 10:56:49'
tags:
- azure
- arm
- troubleshooting
---
I have recently added tasks in to our Azure DevOps ARM template deployment pipeline to run the new `-WhatIf` parameter on the `New-AzResourceGroupDeployment` to preview the changes an ARM deployment will make, per this guide:

- https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-deploy-what-if?tabs=azure-powershell

![ARM deployment whatif example](/content/images/2020/09/resource-manager-deployment-whatif-change-types.png)

This week I discovered that my ARM template validation tasks were no longer working in my pipeline. This was curious because I hadn't changed the script or ARM template since it last ran successfully. The error being returned was:

> InvalidTemplate - Deployment template validation failed: 'Template parameter JToken type is not valid. Expected
'String, Uri'. Actual 'Object'. Please see https://aka.ms/resource-manager-parameter-files for usage details.'.

![Jtoken type is not valid](/content/images/2020/09/Jtoken-type-is-not-valid-error.png)

I eventually discovered this issue:

- https://github.com/Azure/azure-powershell/issues/12792

Which indicates the problem is due to a bug in the Az module and how it handles `SecureString` typed inputs. 

The bug has apparently been fixed in version 4.7.0 of the Az module, but if you're using the Azure DevOps hosted agent "vs2017-win2016" then that version of Az is not currently available (as of 29th Sept 2020). The workaround is therefore to rollback to using an older version of the Az module. After a bit of trial and error the version that worked for me was 4.3.0. Simply set the "Preferred Azure PowerShell Version" setting on your Azure PowerShell tasks to this and you should no longer experience the bug.

Obviously once 4.7.0 is available on the agent image you can revert to using the latest Az version again.