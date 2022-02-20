---
title: Changelog Driven Deployments
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2022/02/change-med.jpg"
  teaser: "/content/images/2020/09/change-sm.jpg"
date: '2022-02-20 09:00:00'
tags:
- powershell
- azuredevops
- cicd
---

A [changelog](https://keepachangelog.com/en/1.0.0/) is a useful addition to any project, as it provides users and contributors with a summary of notable changes between each release. You can ensure you always update your changelog as part of any new release by making it part of the deployment process. This blog post describes how I've done that for each of the PowerShell modules I maintain in GitHub.

I have a standard set of CI/CD scripts that I use as part of my PowerShell modules. 