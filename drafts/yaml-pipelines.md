---
title: Converting Azure DevOps Classic Release deployment pipelines to YAML
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/optic-cables.jpg"
  teaser: "/content/images/2024/optic-cables.jpg"
date: '2024-03-24 09:00:00'
tags:
- powershell
- azure
- azuredevops
- pipelines
- YAML
---

I recently migrated some Azure DevOps Classic Release deployment pipelines to YAML. There's obvious benefits to storing your pipelines as code: they become an artifact in source control that can evolve and change as the code they build or deploy does, and you have the benefits of version history and maintaining the pipelines via pull requests. However I also found that I could use logic and expressions to make the pipelines more efficient and easier to maintain and that through templating could easily connect the pipelines together to form what I humorously dubbed the "super pipeline" (but then the name stuck). In this blog post I will explain the approach I took and the advantages/disadvantages I discovered.

> Maybe the real treasure was the automation improvements we made along the way.

Originally Azure DevOps featured two sections for building pipeline automation, named Builds and Releases. Several years ago Microsoft renamed the Builds section to just "Pipelines" to better reflect that they can be used to create automation for any purpose (including builds or releases), or if you're doing CI/CD, a single pipeline that both builds and releases. Pipelines that you create under "Pipelines" can still be built via the UI, or they can be stored as one or more YAML files in source control. In contrast, pipelines built under "Releases" can only be configured via the Azure DevOps UI. These pipelines do feature a version history, but its only visible within Azure DevOps.

## Getting started

Getting started with YAML pipelines can be a little intimidating. If you go to Pipelines (underneath Pipelines..) and click the "New Pipeline" button in the top right, Azure DevOps will take you through a sort of setup wizard (because Microsoft 😝). You'll first need to select where your code is, which can be an Azure DevOps repository, Github repository, bitbucket cloud or several other option (it's nice that you can use Azure DevOps to build or automate code that lives elsewhere). Once you've picked a location, its going to ask you if you have an existing pipeline definition (i.e a YAML file that is already in the repository) or if you want to create a new one. You can choose "starter pipeline" to have a very basic scaffolding at the top, or choose from a number of templates for different build purposes (it will suggest templates based on the kind of code it finds in the repository you selected). But all of this is very build focussed, and there's nothing overly useful here for someone looking to create a deployment pipeline.

> If you’re converting a Classic Build pipeline to YAML, the process is relatively simple as you can export the whole pipeline as YAML via one step. See the [official guidance here](https://learn.microsoft.com/en-us/azure/devops/pipelines/migrate/from-classic-pipelines?view=azure-devops)
>
> Unfortunately there’s not a built-in way to do the same for Classic Release pipelines. This third party tool can apparently do it, but I didn’t personally try it so use at your own risk:
>
> - [https://github.com/f2calv/yamlizr](https://github.com/f2calv/yamlizr)

Personally I started from scratch, so I suggest finding a suitable location in whatever repository you want to store your pipeline, and create a new file there called (for example) `Deployment.yml` (or whatever you want to name it).

> Working with YAML pipelines can be fiddly, because YAML requires things be indented correctly. I recommend using Visual Studio Code, and installing the [Azure Pipelines Extension](https://marketplace.visualstudio.com/items?itemName=ms-azure-devops.azure-pipelines) as well as an extension to help with formatting YAML, such as [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
>
> If you're new to YAML pipelines I also recommend reading the [Azure DevOps pipelines documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops), to familiarise yourself with things such as the [key concepts](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops).

The Classic Release pipelines I migrated were separated into stages, with a single stage per environment. We would then create a Release, which would consume the artifacts from several builds that contained the built version of the product being deployed, and manually trigger each environment stage whenever we were ready to deploy. There's lots of different ways to implement this as YAML, I could create a pipeline per environment, or have pipelines that deploy to multiple environments, but what I decided to do was use [Templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops&pivots=templates-includes), so that the pipeline could be defined once, but have an environment parameter in it to specify which environment was being deployed.

## Parameters

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

## Variable Groups

To create Variable Groups, in Azure DevOps go to Pipelines > Library. You might want to create a variable group called "All environments", which has default values, or values that apply to all environments. On the Classic Release pipelines the equivalent of this would be variables that were Release scoped. You might also want to create variable groups that apply to more than one of your environments, for values that are the same for all environments of that type, for example: "Non-production environments" and "Production environments". Finally you want to create a variable group for each specific environment, for example: "Test", "Staging", "UAT" etc. Within each of your variable groups, create the various environment specific variables that are required for deployment.

> Note that with the Classic Release pipelines, the variables you created on a pipeline would be cloned when you created a Release, and then any changes you made to the variables on that Release would affect it only. With variable groups, the variables are read from the group at time of execution, so bear in mind that any changes you make to the variables that reference your groups will affect old and new deployments alike.

## Variables

Having created your variable groups, you can now reference them in the YAML template, via a `variables` section. The order in which you specify each group is important, if the same named variable is in multiple groups the last one defined will be used. So define them in priority order. For example:

{% raw %}
```yaml
variables:
  - group: "All environments"

  - ${{ if in(parameters.Environment, 'Test','Staging') }}:
      - group: "Non-production environments"

  - ${{ if in(parameters.Environment, 'UAT', 'Prod') }}:
      - group: "Production environments"

  - group: ${{ parameters.Environment }}
```
{% endraw %}

In the above I've used a [conditional expression](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/expressions?view=azure-devops#conditional-insertion) to define the inclusion of the prod/non-prod groups based on the Environment parameter matching one or more specified names. And then I just use the Environment parameter value itself to include the variable group that I created for the specified environment. The [template expression syntax](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/template-expressions?view=azure-devops) {% raw %}`${{ }}`{% endraw %} is used to include these bits of logic. These get processed when the pipeline is initialised.

## Resources

Because we have separate build pipelines (that generate our deployment artifacts such as infrastructure templates and a compiled release of the software) we next define a `resources` section. [Resources](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=azure-devops&tabs=schema) are anything used by a pipeline that live outside the pipeline. This can include other Azure DevOps pipelines as well as builds from other external CI systems.

In my case, I need to consume some other Azure DevOps pipelines, for example one that builds my infrastructure artifacts (such as ARM templates and deployment scripts) and one that builds the product itself. Because these are Azure DevOps pipelines, we consume them by specifying `pipelines`. The value given for `pipeline` is the alias I want to refer to them via later in the code, `project` is the name of the Azure DevOps project they exist in, `source` is the actual name of the pipeline, and `version` is the specific build version I want to consume.

```yaml
resources:
  pipelines:
    - pipeline: "InfraBuild"
      project: "MyProject"
      source: "Build_Our_Infrastructure"
      version: 1.0.123

    - pipeline: "ProductBuild"
      project: "MyProject"
      source: "Build_Our_Product"
      version: 1.0.456
```

## Trigger

The next thing I define in my YAML pipeline is the `trigger`. By default pipelines trigger whenever there is a commit to any branch in their respective repository, but because this is a deployment pipeline that I want to trigger manually, we need to set trigger to none. 

```yaml
trigger: none
```

## Stages

We're now ready to start defining the actual tasks the deployment pipeline will perform. You don't have to use stages as part of this, but by grouping the different associated sets of tasks into stages you can then easily enable or disable them at the point where you run the deployment. For example you might have a pipeline that deploys databases, configuration and then the application/s. It might make sense to split these into stages if you might have a run in the future where you only want to execute a subset of them. 

To define stages we need to specify `stages:` and then under that each stage with a name. Within the stage we can define `displayName` to have a more descriptive title appear in the UI when the pipeline is run. You use `pool` to define which of your Azure DevOps deployment pools will execute this task. You can define this at the top level if all of your tasks will run under the same pool, but for my pipeline I wanted some tasks to run via a self hosted agent and some tasks to run via Microsoft-Hosted agents, so it was necessary to define it at the stage level. 

Within each Stage you can have one or more `jobs`. We use a special job type here called `deployment` to indicate to Azure DevOps that this is a [deployment job](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs?view=azure-devops) we're running. Under deployment you specify `environment` which then allows you to view your deployments per environment under the Pipelines > Environments section of Azure DevOps. In the example below i'm using the `Environments` parameter value to populate this via the expression syntax. We then need to specify `strategy` because this is a deployment job. For my purpose this needs to be `runOnce` because I'm just deploying to a single environment and I just want each phase of the deployment to run once. You can split your tasks under different phases (predeploy, deploy, routeTraffic and postRouteTraffic) but I personally just have all my tasks under deploy. Finally you use `steps:` under which you then define each of the tasks to be performed.  

Here's an example first stage, with some tasks for unzipping and deploying a database DACPAC:

{% raw %}
```yaml
stages:
  - stage: DatabaseDeployment
    displayName: "Deploy Database"
    pool: "My Deployment Pool"
    jobs:
      - deployment: DatabaseDeployment
        environment: ${{ parameters.Environment }}
        strategy:
          runOnce:
            deploy:
              steps:
                - download: ProductBuild
                  artifact: Database
                  displayName: Download Database Release

                - task: ExtractFiles@1
                  displayName: "Extract Database zip file"
                  inputs:
                    archiveFilePatterns: 'ProductBuild\Database\Database.zip'
                    destinationFolder: 'ProductBuild\Database'
                    cleanDestinationFolder: false

                - task: AzureKeyVault@1
                  displayName: "Get the Database password from KeyVault"
                  inputs:
                    azureSubscription: "$(azureSubscription)"
                    KeyVaultName: "$(KeyVaultName)"
                    SecretsFilter: DatabasePassword

                - task: SqlAzureDacpacDeployment@1
                  displayName: "Deploy Product Database"
                  inputs:
                    azureSubscription: "$(azureSubscription)"
                    ServerName: "productdb-$(EnvironmentName).database.windows.net"
                    DatabaseName: ProductDatabase
                    SqlUsername: databaseadmin
                    SqlPassword: "$(DatabasePassword)"
                    DacpacFile: "ProductBuild/Database/ProductDatabase.dacpac"

```
{% endraw %}

Notice that the first task here is downloading the required artifact from one of the resources I specified in `resources:`. By default this task would download all artifacts from the resource, but if you know for a specific stage that you only need a subset of them (and assuming your build outputs multiple resources), then you can specify just the ones you need to speed things up.

If you're converting a Classic Release pipeline to YAML, you can get the YAML for your individual tasks by going to your Classic Release pipeline and the Tasks view. Then for each individual task there is a "View YAML" link in the top right. You can copy this and paste it into your YAML file, ensuring you indent it appropriately. If the task references a different resource name alias than the one you configured in the `resources:` section you'll need to update that. It will also add comments to the top of the YAML it produces warning you of any variables that have been used that you'd then need to either define on the pipeline directly (under `variables:`) or via the variable groups I suggested earlier. 

You can see in my example tasks above that I reference a `$(azureSubscription)` variable, that is the name of my Azure Subscription. This would be the sort of variable that would exist in my Non-production environments and Production environments variable groups, as I'd have a separate subscription for each. And then i've got `$(KeyVaultName)` and `$(EnvironmentName)` variables, these would be environment specific values that would exist in each of my environment variable groups. The `$()` syntax is the same variable substitution syntax that is used on the Classic Release pipelines. Note that variables using this syntax are populated when the pipeline runs.

## Dependencies

The stages / jobs / tasks that you specify in a YAML pipeline will execute in the order that you list them, but you can use the `dependsOn` setting to specify that certain stages depend on the execution of one or more previous ones. This is also how you can specify certain stages to execute in parallel. For example, imagine I had my DatabaseDeployment stage above, and then I had stages called ConfigurationDeployment and ApplicationDeployment. Lets say my configuration gets written into the database, and my application will just keep polling for it if until it exists. So to speed up my deployment i'm happy for my configuration and application to deploy in parallel, so long as the database has been deployed. By using `dependsOn` these two stages will run in parallel:

{% raw %}
```yaml
  - stage: ConfigurationDeployment
    displayName: "Deploy Configuration"
    pool: "My Deployment Pool"
    dependsOn: DatabaseDeployment
    jobs:
      - deployment: ConfigurationDeployment
        environment: ${{ parameters.Environment }}
        strategy:
          runOnce:
            deploy:
              steps:
                - ...

  - stage: ApplicationDeployment
    displayName: "Deploy Application"
    pool: "My Deployment Pool"
    dependsOn: DatabaseDeployment
    jobs:
      - deployment: ApplicationDeployment
        environment: ${{ parameters.Environment }}
        strategy:
          runOnce:
            deploy:
              steps:
                - ...
```
{% endraw %}

For the tasks to execute in parallel you need to be using a deployment pool that supports more than 1 parallel job. If you're using the free tier Microsoft-hosted agents and a private project you only get 1 job by default. If you purchase parallel jobs you have to purchase at least 2 (once you have a paid job the free one no longer applies).

If you want some tasks to execute in parallel at the beginning of your pipeline, you can specify `dependsOn: []`.

## Repetition

If you have a number of repetitive tasks in your YAML pipeline, you can use the `each` expression to repeat them so you only have to specify them (and therefore maintain them) once. For example, if you need to deploy 3 applications to Azure App Service, you could do something like the following:

First you need a parameter defined to specify the list of items (in this case applications) to deploy:

```yaml
parameters:
  - name: AppsToDeploy
    type: object
    values:
      - "App1"
      - "App2"
      - "App3"
```

Then for your task (or for a set of tasks, or you could do it at the stage level) you specify the `each` expression:

{% raw %}
```yaml
- ${{ each app in parameters.AppsToDeploy }}:
  - task: AzureRmWebAppDeployment@3
    displayName: "Deploy ${{ app }} to App Service"
    inputs:
      azureSubscription: '$(azureSubscription)'
      WebAppName: ${{ app }}
      Package: '/ProductBuild/${{ app }}/${{ app }}.zip'
      SetParametersFile: '/ProductBuild/${{ app }}/${{ app }}.SetParameters.xml'
```
{% endraw %}

If you do this at the stage level, then when you select Run Pipeline in Azure DevOps, the pipeline will be initialised and these stages setup, which if you then select "Stages to run" under Advanced options, you'll see them (and if you only wanted to deploy one of the applications, you could turn one or more of the stages off).

Having the input values as a parameter would mean that you would see (and could modify them) at the the time of triggering the pipeline. However we can hide them away by making the application deployment stages a separate template.

To do this, you would create a new YAML file called something like `ApplicationDeployment-Stages.yml`. This would need to have all the necessary parameters for the applicaiton deployment tasks (for example the `Environment` one, as well as the `AppsToDeploy` one above). But in this template you don't then specify the `resources:`, `variables:` or `trigger:` sections. After the parameters, just specify the various stages, jobs and tasks that apply to App Deployment.

Then back in our `Deployment.yml` template, we can remove all of the application deployment tasks and instead reference the template, along with any input parameters we want to pass along, for example:

```yaml
stages:
  - template: ApplicationDeployment-Stages.yml
    parameters:
      Environment: ${{ parameters.Environment }}
```

The `AppsToDeploy` parameter no longer needs to be present on the `Deployment.yml` file, as within `ApplicationDeployment-Stages.yml` its default value is then always used.

By separating off groups of related stages into separate templates, you can build different combinations of the templates for different purposes. And this where I ended up creating a "super pipeline", as once I had a template for infrastructure deployment, application deployment etc. I could then have a pipeline that referenced each of them to have a single pipeline to do a full deployment (such as when standing up an environment for testing). While still having separate definitions for when I just needed to do either infrastructure or application deployments.

## Conclusion

There's a lot to love about YAML pipelines. As I said in the beginning, having everything defined as code (including your pipelines), is powerful in itself because you get all the benefits of source control when maintaining those code artifacts, and (if you implement templates well) pipelines that are a lot less hassle to maintain. Being able to use the `if` and `each` expressions allows you to build pipelines that are more versatile and easier to execute. With the Classic Release pipelines running a subset of tasks would usually mean editing a release to enable/disable certain tasks. But with the YAML pipelines well defined parameters or stages allow you to implement this flexibility as part of the design. And although you can also do parallel tasks in the Classic Release pipelines its a lot less effort to implement via YAML.

That being said, there's a few downsides as well. If you used the approach of having stages per environment in your Classic Release pipelines, then it was very easy to see which environment was deployed to which release, and you could take a kind of left to right deployment approach. Although there is the Environments pane within Azure DevOps that gets populated when you use a `Deployment` type job, its not as easy to see the current status of your environments against each other. One way in which I made this a little more transparent was to implement the `name:` setting in my pipelines, to customise the run name to include the environment and the version of the product being deployed. For example:

```yaml
name: Application Deployment - ${{ parameters.Environment }} - 1.0.123 - Run $(Rev:r)
```

The other downside is with the variables. As I noted earlier, the variable groups are very similar to how you'd configure variables on the release pipelines, but there's no concept of creating a release that then stamps a version of those variables. In a way this is a positive as well, because maintaining variables on the Classic Release pipelines can be a bit of a nightmare when they're copied all over the place, but if you make some changes to your variable groups for the next release, and those changes weren't appropriate for the previous release then you've broken some backwards compatibility. There's ways to workaround this but it requires thought and careful consideration. 

Personally I found the positives outweighed the negatives and ended up with some easy to maintain pipelines that reduced the deployment time by multiple hours, and allowed me to build the "super pipeline" so I can one click deploy test environments. I also built an environment removal pipeline to have a one click tear down. Which means costs saved, by making it easy to run and remove environments only when required.