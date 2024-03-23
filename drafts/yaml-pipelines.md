---
title: Converting Azure DevOps Classic Release deployment pipelines to YAML
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

I recently migrated some Azure DevOps Classic Release deployment pipelines to YAML. There's obvious benefits to storing your pipelines as code: they become an artifact in source control that can evolve and change as the code they build or deploy does, and you have the benefits of version history and maintaining the pipelines via pull requests. However I also found that I could use logic and expressions to make the pipelines more efficient and easier to maintain and that through templating could easily connect the pipelines together to form what I dubbed the "super pipeline". In this blog post I will explain the approach I took to this migration and the benefits and downsides I discovered along the way.

> Maybe the real treasure was the automation improvements we made along the way.

Originally Azure DevOps featured two sections for building pipeline automation, named Builds and Releases. Several years ago Microsoft renamed the Builds section to just "Pipelines" to better reflect that they can be used to create automation for any purpose (including builds or releases), or if you're doing CI/CD, a single pipeline that both builds and releases. Pipelines that you create under "Pipelines" can still be built via the UI, or they can be stored as one or more YAML files in source control. In contrast, pipelines built under "Releases" can only be configured via the Azure DevOps UI. These pipelines do feature a version history, but its only visible within Azure DevOps.

## Getting started

Getting started with YAML pipelines can be a little intimidating. If you go to Pipelines (underneath Pipelines..) and click the "New Pipeline" button in the top right, Azure DevOps will take you through a sort of setup wizard (because Microsoft :P). You'll first need to select where your code is, which can be an Azure DevOps repository, Github repository, bitbucket cloud or several other option (it's nice that you can use Azure DevOps to build or automate code that lives elsewhere). Once you've picked a location, its going to ask you if you have an existing pipeline definition (i.e a YAML file that is already in the repository) or if you want to create a new one. You can choose "starter pipeline" to have a very basic scaffolding at the top, or choose from a number of templates for different build purposes (it will suggest templates based on the kind of code it finds in the repository you selected). But all of this is very build focussed, and there's nothing overly useful here for someone looking to create a deployment pipeline.

> If you’re converting a Classic Build pipeline to YAML, the process is relatively simple as you can export the whole pipeline as YAML via one step. See the [official guidance here](https://learn.microsoft.com/en-us/azure/devops/pipelines/migrate/from-classic-pipelines?view=azure-devops)
>
> Unfortunately there’s not a built-in way to do the same for Classic Release pipelines. This third party tool can apparently do it, but I didn’t personally try it so use at your own risk:
>
> - [https://github.com/f2calv/yamlizr](https://github.com/f2calv/yamlizr)

Personally I started from scratch, so I suggest finding a suitable location in whatever repository you want to store your pipeline, and create a new file there called (for example) `Deployment.yml` (or whatever you want to name it).

> Working with YAML pipelines can be fiddly, because YAML requires things be indented correctly. I recommend using Visual Studio Code, and installing the [Azure Pipelines Extension](https://marketplace.visualstudio.com/items?itemName=ms-azure-devops.azure-pipelines) as well as an extension to help with formatting YAML, such as [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

The Classic Release pipelines I migrated were separated into stages, with a single stage per environment. We would then create a Release, which would consume the artifacts from several builds that contained the built version of the product being deployed, and manually trigger each environment stage whenever we were ready to deploy. There's lots of different ways to implement this as YAML, I could create a pipeline per environment, or have pipelines that deploy to multiple environments, but what I decided to do was use [Templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops&pivots=templates-includes), so that the pipeline could be defined once, but have an environment parameter in it to specify which environment was being deployed.

To specify parameters, your YAML file needs a parameters section, under which you create one or more parameters by specifying a name, type and default value. If you want the parameter to be a drop-down selection, you can specify a list of accepted values:

```yaml
parameters:
  - name: Environment
    type: string
    values:
      - "Test"
      - "Staging"
      - "UAT"
      - "Production"

    default: "Test"
```

When the pipeline is executed, the environment selection needs to make sure that we're using variables specific to that environment. Under the Classic Release pipeline, if you took the approach of having a stage per environment, you could then have variables scoped to that environment (stage). To achieve the same result for the YAML pipeline, we can create Variable Groups, which we link to the template.

To create Variable Groups, in Azure DevOps go to Pipelines > Library. You might want to create a variable group called "All environments", which has default values, or values that apply to all environments. On the Classic Release pipelines the equivalent of this would be variables that were Release scoped. You might also want to create variable groups that apply to more than one of your environments, for values that are the same for all environments of that type, for example: "Non-production environments" and "Production environments". Finally you want to create a variable group for each specific environment, for example: "Test", "Staging", "UAT" etc. Within each of your variable groups, create the various environment specific variables that are required for deployment.

> Note that with the Classic Release pipelines, the variables you created on a pipeline would be cloned when you created a Release, and then any changes you made to the variables on that Release would affect it only. With variable groups, the variables are read from the group at time of execution, so bear in mind that any changes you make to the variables that reference your groups will affect old and new deployments alike.

Having created your variable groups, you can now reference them in the YAML template, via a `variables` section. The order in which you specify each group is important, if the same named variable is in multiple groups the last one defined will be used. So define them in priority order. For example:

{% raw %}
```yaml
variables:
  - group: "All environments"

  - ${{ if in(parameters.Environment, 'Test','Staging') }}:
      - group: "Non-production environments"

  - ${{ if in(parameters.Environment, 'UAT', 'Prod') }}:
      - group: "Production environments"

  - group: ${{ parameters.Environment }}```
```
{% endraw %}

In the above I've used a [conditional expression](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/expressions?view=azure-devops#conditional-insertion) to define the inclusion of the prod/non-prod groups based on the Environment parameter matching one or more specified names. And then I just use the Environment parameter value itself to include the variable group that I created for the specified environment. The [template expression syntax](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/template-expressions?view=azure-devops) `${{ }}` is used to include these bits of logic. These get processed when the pipeline is initialised.



```yaml
resources:
  pipelines:
    - pipeline: "YourPipeline"
      project: "YourProject"
      source: "Name_Of_Your_Pipeline"
      version: 2.15.47
```



```yaml
trigger: none

```

## Resources



## Parameters



## Stages



## DependsOn



## Repetition



## Automatic Retries



## Variables



## Secure Files



## Advantages/Disadvantages

