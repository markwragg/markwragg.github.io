---
title: Converting Azure DevOps Classic Pipelines to YAML
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/optic-cables.jpg"
  teaser: "/content/images/2024/optic-cables.jpg"
date: '2024-03-01 09:00:00'
tags:
- powershell
- azure
- azuredevops
- pipelines
- YAML
---

 I've spent some time recently recreating some Classic Release pipelines as YAML. My main motivation was to reduce deployment times by having some tasks run in parallel. It is possible to do this via the Classic Release pipelines, but if you have deployments that are complex then things can quickly get unwieldy. In this blog post I will explain the approach I took to migrating some pipelines to YAML and the benefits I discovered along the way.

> Maybe the real treasure was the automation benefits we discovered along the way.

Originally Azure DevOps featured two sections for building pipeline automation: Builds and Releases. Several years ago Microsoft renamed the Builds section to just "Pipelines" to better reflect that they can be used to create automation for any purpose (including builds or releases), or if you're doing Continuous Deployment, a single pipeline that both builds and releases. Pipelines that you build under "Pipelines" are stored as one or more YAML files in source control. In contrast, pipelines built under "Releases" are configured via the Azure DevOps UI. They do feature a version history, but they are stored within Azure DevOps itself.

## Getting started

Getting started with YAML pipelines can be a little intimidating. If you go to Pipelines (underneath Pipelines..) and click the "New Pipeline" button in the top right, Azure DevOps will take you through a sort of setup wizard (because Microsoft :P). You'll first need to select where your code is, which can be an Azure DevOps repository, Github repository, bitbucket cloud or several other options. It's nice that you can use Azure DevOps to build or automate code that lives elsewhere. Once you've picked a location, its going to ask you if you have an existing pipeline definition (i.e a YAML file that is already in the repository) or if you want to create a new one. You can choose "starter pipeline" to have a very basic scaffolding at the top, or from a number of other templates aimed at various specific purposes. 

> If you're converting a Classic Build pipeline to YAML, the process is relatively simple as you can export the whole pipeline as YAML via one step. See the official guidance here:
>
> - https://learn.microsoft.com/en-us/azure/devops/pipelines/migrate/from-classic-pipelines?view=azure-devops
>
> Unfortunately there's not a built-in way to do the  same for Classic Release pipelines. This third party tool can do it, but I didn't personally try it so use at your own risk:
>
> - https://github.com/f2calv/yamlizr



## Resources



## Parameters



## Stages



## DependsOn



## Repetition



## Automatic Retries



## Variables



## Secure Files



## Advantages/Disadvantages

