---
title: Local ARM template deployments using Azure DevOps variables
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2022/03/keyboard-code-lg.jpg"
  teaser: "/content/images/2022/03/keyboard-code-sm.jpg"
date: '2022-03-01 14:00:00'
tags:
- powershell
- azuredevops
- ARM
- deployment
draft: true
---

# Problem

When working with ARM templates its good practice to use deployment pipelines to deliver the changes into your environments. However when developing your changes, testing via a pipeline can be slow because it requires the creation of builds and releases. Instead you want to be able to trigger the deployment of your changes to a test environment from your local machine.

Deploying to your environments from your local machine can be difficult if your environment specific config is loaded as part of the deployment from Azure DevOps variables, calls to Azure resources and/or secrets from KeyVault.

# Solution 

To enable deployment from a local machine, I've written a script that pulls the configuration from Azure DevOps and then feeds it into a PowerShell ARM deployment as template inputs.

Different locations inputs can come from:

- Direct input on the script
- Parameters file
- Azure DevOps variable for the stage
- Azure DevOps variable groups
- Azure DevOps Release variables
- Directly from Azure Resources (such as storage account keys)
- Secrets in KeyVaults


