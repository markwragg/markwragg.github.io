---
title: Local ARM template deployments using Azure DevOps variables
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2022/02/keyboard-code-lg.jpg"
  teaser: "/content/images/2022/02/keyboard-code-sm.jpg"
date: '2022-03-01 14:00:00'
tags:
- powershell
- azuredevops
- ARM
- deployment
---

When working with ARM templates its good practice to use deployment pipelines to deliver the changes into your environments. However when developing your changes, testing via a pipeline can be slow because it requires the creation of builds and releases. Instead you want to be able to trigger the deployment of your changes to a test environment from your local machine.