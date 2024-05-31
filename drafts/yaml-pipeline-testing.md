---
title: Testing Azure DevOps YAML pipelines with Pester
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/Pipeline-testing.jpeg"
  teaser: "/content/images/2024/Pipeline-testing.jpeg"
date: '2024-03-24 09:00:00'
tags:
- powershell
- pester
- azure
- azuredevops
- pipelines
- YAML
---

In my [previous blog post](https://wragg.io/converting-azure-devops-classic-release-pipelines-to-yaml/) I covered how I recently converted a number of Classic Azure Deployment pipelines to YAML. In this blog post I will explain how I then authored some PowerShell Pester tests as a way to validate the pipelines are configured and working as expected.

