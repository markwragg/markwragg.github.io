---
title: Well-architect your Azure Terraform with PSRule
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/one-ring.jpg"
  teaser: "/content/images/2025/one-ring.jpg"
date: "2025-03-02 09:00:00"
tags:
  - azure
  - psrule
  - terraform
  - azuredevops
  - pipelines
  - powershell
---

This blog post is part of the [Azure Spring Clean 2025](https://learn.microsoft.com/en-us/azure/well-architected/) community event, promoting well managed Azure tenants. In last year's Azure Spring Clean, [Dan Rios blogged about using PSRule for Bicep code](https://rios.engineer/azure-spring-clean-azure-best-practice-for-bicep-with-psrule/). The focus of this blog post is on how you can use PSRule to validate Azure resources deployed via [Terraform by HashiCorp](https://www.terraform.io/).

> Many DevOps tools were gifted to the Engineers, who above all else, desired automation. For within these tools was bound the strength and will to govern the Cloud.
> But they were all of them deceived, for another tool was made.
> In the land of Microsoft, in the fires of Mount Azure, the Architect [Bernie White](https://www.linkedin.com/in/bernie-white/) forged, in open source, a tool to validate all others.
> And into this tool he poured his creativity, his mastery and his will to improve all infrastructure.
>
> One tool to [PSRule](https://microsoft.github.io/PSRule/v2/) them all.

I suspect it was something like that anyway.

[PSrule can be found in GitHub](https://github.com/microsoft/PSRule) and has been in development since December 2018. The current stable version is v2.9.0 (v3 is in development). The [official website](https://microsoft.github.io/PSRule/v2/) has lots of guidance on getting started, and is officially described as:

> "A rules engine geared towards testing Infrastructure as Code (IaC). Rules you write or import perform static analysis on IaC artifacts such as: templates, manifests, pipelines, and workflows."

The idea is that you (or Microsoft, the community, the Council of Elrond) define rules for how your infrastructure should be configured and PSRule (executed as part of your development process) confirms those constraints are being followed. Infrastructure code is often complex, and can be developed by multiple individuals or teams over time. PSRule can act as a guard rail to ensure the infrastructure requirements of your organisation are being followed, or used to evaluated the quality of existing infrastructure.

Obviously developing these rules is itself a timely endeavour, but PSRule has done the heavy lifting for you by providing various pre-built rules based on best practice guidance such as the [Azure Well-architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/). PSRule is extensible, so you choose which existing rules you want to use, and customise them (and/or develop your own rules) to meet your requirements.

### Begin the journey

> _"It's a dangerous business, Frodo, going out your door."_ — Bilbo Baggins

[Installing PSRule for Azure](https://azure.github.io/PSRule.Rules.Azure/install/) requires no difficult journey to Mount Doom. There is a [dedicated site for this module](https://azure.github.io/PSRule.Rules.Azure/), with its own [Getting started](https://azure.github.io/PSRule.Rules.Azure/about/) page. You can install it directly, as follows:

- [Install PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell) (if you haven't already)
- Install PSRule (with the pre-built Azure rules) from the PowerShell Gallery:

```powershell
Install-Module -Name 'PSRule.Rules.Azure' -Repository PSGallery -Scope CurrentUser
```

PSRule for Azure is officially described as:

> "A pre-built set of tests and documentation to help you configure Azure solutions. These tests allow you to check your Infrastructure as Code (IaC) before or after deployment to Azure. PSRule for Azure includes unit tests that check how Azure resources defined in **ARM templates or Bicep code** are configured."

As it turns out, one does not _simply_ test Terraform with PSRule.

![One does not simply test Terraform meme](/content/images/2025/one-does-not-simply-test-terraform.jpg){: .align-center}

But it's not _that_ complicated. At the moment you can't perform static analysis of your Terraform files (or Terraform plan output), but you can [indicate your support for the feature here](https://github.com/Azure/PSRule.Rules.Azure/issues/1193). For now, you need to deploy resources to Azure, run a command to export the configuration of those resources to a JSON file, which PSRule can then analyse. While this doesn't make PSRule as useful as it is for analysing Bicep (for which there is also a [VSCode extension](https://marketplace.visualstudio.com/items?itemName=bewhite.psrule-vscode) so you can get feedback while developing), if you deploy your infrastructure through a series of environments for testing (or even if you don't) I can see PSRule still forming a valuable part of that pipeline.

To summarise, here's the steps to take to analyse Terraform resources:

1. Deploy your Terraform code to Azure (ideally to a non-production environment).
2. Ensure you have authenticated to Azure via `Connect-AzAccount` and have the relevant subscription selected.
3. Export your Azure resources to a specified directory (and ensure the directory exists):

```powershell
New-Item -Path out -ItemType Directory
Export-AzRuleData -OutputPath "$pwd/out"
```

By default this will export all of the resources for the currently selected subscription. You can use `-Subscription` to specify one or more subscriptions, or `-ResourceGroupName` to specify one or more resource groups. You can also use `-Tag` to export resources with one or more specified tags.

If you want to go big (rather than return to the shire), you can use `-All` to export resources from all the subscriptions your current context can access.

The output file/s are named with the guid of the subscription. This is true even if you filter to specific resources by Resource Group or Tag, and any existing file with the same name will be overwritten.

### Face your demons

> _"It is the small things, everyday deeds of ordinary folk that keep the darkness at bay."_ — Gandalf

It's now time to analyse the output. You do this with `Invoke-PSRule` which you need to point at your export file/s and the `PSRule.Rules.Azure` module:

```powershell
Invoke-PsRule -InputPath "$pwd/out/" -Module 'PSRule.Rules.Azure'
```

Depending on your resources, you'll probably get quite a lot of output because by default the tool returns all results (pass, fail and error). If you want to just see failed results, execute:

```powershell
Invoke-PSRule -InputPath "$pwd/out/" -Module 'PSRule.Rules.Azure' -Outcome Fail
```

![PSRule for Azure output example](/content/images/2025/psrule-output.png){: .align-center}

Results from PSRule are returned as PowerShell objects, so it's worthwhile returning them to a variable as there are more properties returned than you see in the default view above. But what you do get by default is the rule outcomes grouped by resource, with the name of the rule and a further description of how to address it in the Recommendation.

Further properties in the object that are useful include:

- Ref — This is the unique reference number for the rule. You can use this or the rule name to get to the relevant [reference page](https://azure.github.io/PSRule.Rules.Azure/en/rules/) which has more detail explaining the rule and sometimes examples of how to address it.
- Reason — This is useful to get more specific detail of the violation, particularly if it relates to one or more sub-resources.

You also might want to export the results for some further/offline analysis and PSRule makes this easy with it's `-OutputPath` parameter, along with `-OutputFormat` that can be used to output to various formats such as CSV, JSON, Markdown, YAML and NUnit3.

There are some other switches on the command that are worth exploring, one of which is `-As Summary` which simply returns a table showing for each rule how many resources passed and failed:

```plaintext
RuleName                            Pass  Fail  Outcome
--------                            ----  ----  -------
Azure.AppService.PlanInstanceCount  0     4     Fail
Azure.AppService.MinTLS             20    0     Pass
Azure.KeyVault.Logs                 19    0     Pass
Azure.KeyVault.Name                 19    0     Pass
Azure.Resource.UseTags              153   0     Pass
Azure.Resource.AllowedRegions       154   0     Pass
Azure.ResourceGroup.Name            20    0     Pass
Azure.ServiceBus.Usage              4     0     Pass
Azure.SQL.FirewallRuleCount         1     1     Fail
Azure.SQL.AllowAzureAccess          0     2     Fail
Azure.Storage.UseReplication        23    6     Fail
Azure.Storage.SoftDelete            2     12    Fail
Azure.RBAC.UseRGDelegation          20    0     Pass
Azure.VM.PublicIPAttached           5     1     Fail
Azure.VNET.UseNSGs                  0     4     Fail
...
```

### Make the hard choices (or easy ones)

> _"This task was appointed to you. And if you do not find a way, no one will."_ — Galadriel

Once you have the output there's obviously two ways forward (three if you count doing nothing):

1. Address the failure by modifying your Terraform code
2. Exclude the rule

If there are issues that you can and want to fix, then simply make the required changes in your Terraform code (using the guidance provided by the recommendation). Redeploy your infrastructure and re-run the export of the rule data and analysis to confirm the test now passes.

There may be issues that you don't want to fix, because the supposed misconfiguration is appropriate for your infrastructure (for example, you might have a Storage account that you need to be publicly accessible) or there may be entire rulesets that you don't consider in scope for your infrastructure (you may have no need for tags for example). PSRule allows you to control these exceptions by [configuring exclusion or suppression](https://azure.github.io/PSRule.Rules.Azure/concepts/suppression/).

These customisations are configured by creating a file called `ps-rule.yaml`.

You can exclude a rule as follows:

```yaml
rule:
  exclude:
    # Ignore the following rules for all resources
    - Azure.Resource.UseTags
```

And you can suppress a rule for specific resources like this:

```yaml
suppression:
  Azure.Storage.BlobPublicAccess:
    # Ignore blob public access on the following storage account
    - mypublicstorageaccount
```

These rules will no longer pass or fail.

![You shall not pass or fail rule is excluded meme](/content/images/2025/you-shall-not-pass.jpg){: .align-center}

There's some further cleverness you can do to avoid having to manually populate resources in these files, by instead creating logic that sets up exclusions based on the value of fields such as names or tags. These are called [suppression groups](https://microsoft.github.io/PSRule/v2/concepts/PSRule/en-US/about_PSRule_SuppressionGroups/).

The built-in rules are routinely updated (usually monthly). As the rules are updated, the output you get may change. You can manage this by [running PSRule against a specified baseline](https://azure.github.io/PSRule.Rules.Azure/working-with-baselines/). This keeps your testing consistent, until you decide to move to a new baseline. There are also pillar specific baselines, for example if you're only interested using PSRule to evaluate cost optimisation you can use the `Azure.Pillar.CostOptimization` baseline. You can also [create your own custom baselines](https://microsoft.github.io/PSRule/v2/concepts/PSRule/en-US/about_PSRule_Baseline/).

[Available baselines are listed here](https://azure.github.io/PSRule.Rules.Azure/en/baselines/). To run PSRule against a specified baseline:

```powershell
Invoke-PSRule -InputPath "$pwd/out/" -Module 'PSRule.Rules.Azure' -Outcome Fail -Baseline 'Azure.GA_2024_09'
```

You will see a warning when a specified baseline is outdated.

### Don't go it alone

> _"Help me bear this burden."_ — Frodo

While it might be helpful to have a band of [hobbitses](https://en.wiktionary.org/wiki/hobbitses) to assist with this work, a more DevOpses approach might be to implement a pipeline. Once you've done the above: established your baseline and excluded or suppressed rules that do not apply to your infrastructure, it may make sense to include a run of PSRule as part of your CI pipeline, and perhaps as a gate for infrastructure Pull Requests. [PSRule provides guidance on this](https://azure.github.io/PSRule.Rules.Azure/creating-your-pipeline) but it is geared towards testing the Bicep or ARM static files. For Terraform we need to do an in-flight analysis post-deployment.

Here's how you might do that in Azure DevOps:

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Plan
  displayName: 'Terraform Plan'
  jobs:
  - job: TerraformPlan
    displayName: 'Terraform Plan'
    steps:
    - task: TerraformInstaller@0
      inputs:
        terraformVersion: 'latest'

    - script: |
        terraform --version
        terraform init -backend-config="storage_account_name=$(storageAccountName)" -backend-config="container_name=$(containerName)" -backend-config="key=$(key)" -backend-config="access_key=$(accessKey)"
        terraform plan -out=tfplan
      displayName: 'Terraform Init and Plan'

- stage: Deploy
  displayName: 'Terraform Apply'
  jobs:
  - job: TerraformApply
    displayName: 'Terraform Apply'
    steps:
    - task: TerraformInstaller@0
      inputs:
        terraformVersion: 'latest'

    - script: |
        terraform --version
        terraform init -backend-config="storage_account_name=$(storageAccountName)" -backend-config="container_name=$(containerName)" -backend-config="key=$(key)" -backend-config="access_key=$(accessKey)"
        terraform apply -auto-approve tfplan
      displayName: 'Terraform Init and Apply'

- stage: PSRule
  displayName: 'Execute PSRule'
  jobs:
  - job: PSRule
    displayName: 'Execute PSRule'
    steps:
      - task: AzurePowerShell@5
            displayName: Export PSRule Data and Execute Rules
            inputs:
              azureSubscription: $(azureSubscription)
              scriptType: InlineScript
              azurePowerShellVersion: LatestVersion
              Inline: |
                Install-Module -Name 'PSRule.Rules.Azure' -Scope CurrentUser -Force -ErrorAction Stop

                New-Item -Path out -ItemType Directory -Force
                Export-AzRuleData -OutputPath 'out'

                Assert-PSRule -InputPath 'out' -Module 'PSRule.Rules.Azure' -Outcome Fail -Baseline 'Azure.GA_2024_12'
```

Note the usage of `Assert-PSRule` instead of `Invoke-PSRule`. This command returns formatted text instead of PowerShell objects which is easier to read in the output of a pipeline job.

The pipeline job will fail if the tests fail, and the output will look something like this:

![PSRule for Azure pipeline output example](/content/images/2025/psrule-pipeline-output.png){: .align-center}

### Journeying onward

> _"Don't adventures ever have an end? I suppose not. Someone else always has to carry on the story."_ — Bilbo Baggins

If you've conquered all of the above, well done! But what comes after? Well you could give some consideration to authoring your own [custom rules](https://azure.github.io/PSRule.Rules.Azure/customization/using-custom-rules/). To do this:

1. Create a `.ps-rule` directory in your repository.
2. Create a new custom rule ps1 file. The rule name is up to you, but must be unique. For example: `Org.Azure.Rule.ps1`.
3. In the file define your rule. For example, here's how you might configure a rule for custom tags:

```powershell
# Synopsis: Resource Groups must have all mandatory tags defined.
Rule 'Org.Azure.RG.Tags' -Type 'Microsoft.Resources/resourceGroups' {
    $hasTags = $Assert.HasField($TargetObject, 'Tags')
    if (!$hasTags.Result) {
        return $hasTags
    }

    # Require tags be case-sensitive
    $Assert.HasField($TargetObject.tags, 'costCentre', $True)
    $Assert.HasField($TargetObject.tags, 'env', $True)
}
```

In the example above the `HasField` assertion takes 3 inputs, the value being evaluated, the expected value and a boolean indicating whether the comparison should be case-sensitive. There are plenty of [other assertions](https://microsoft.github.io/PSRule/v2/concepts/PSRule/en-US/about_PSRule_Assert/) you can use.

Before your rule will work, you also need to add a file to the root of your repository named `ps-rule.yaml` with the following content:

```yaml
# Configure binding options
binding:
  targetType:
    - 'resourceType'
    - 'type'
```

This binding allows custom rules to use the `-Type` parameter. The in-built rules detect this automatically, but custom rules check the `ps-rule.yaml` file for their type binding configuration.

You can test your custom rule by executing `Invoke-PSRule` with a `-Path` parameter:

```powershell
Invoke-PSRule -Path "$pwd/.ps-rule/" -InputPath "$pwd/out/"
```

The output will look something like this:

```plaintext
RuleName                            Outcome    Recommendation
--------                            -------    --------------
Org.Azure.RG.Tags                   Fail

   TargetName: sometargetresource
```

You'll notice there's no Recommendation returned. We can add this by adding the `Recommend` keyword into your rule:

```powershell
Rule 'Org.Azure.RG.Tags' -Type 'Microsoft.Resources/resourceGroups' {
    $hasTags = $Assert.HasField($TargetObject, 'Tags')
    if (!$hasTags.Result) {
        return $hasTags
    }

    # Require tags be case-sensitive
    $Assert.HasField($TargetObject.tags, 'costCentre', $True)
    $Assert.HasField($TargetObject.tags, 'env', $True)

    Recommend "The following tags are mandatory: 'costCentre','env'"
}
```

And now we can see the message:

```plaintext
RuleName                            Outcome    Recommendation
--------                            -------    --------------
Org.Azure.RG.Tags                   Fail       The following tags are mandatory: 'costCentre','env'

   TargetName: sometargetresource
```

I hope you've found this a useful introduction to PSRule. While the focus of this blog was on Terraform, most of what is detailed above can be used to test infrastructure deployed via any mechanism (even resources created manually, but please don't :) ).

For other great content in this year's Azure Spring Clean, check out the [website](https://www.azurespringclean.com/) or follow the [Azure Spring Clean list on BlueSky](https://bsky.app/profile/wragg.io/lists/3ljhoyojxk42m).

![Lord of the Rings fellowship](/content/images/2025/lotr-fellowship.jpg){: .align-center}

_Did you know that in the books, 17 years pass between Frodo receiving the ring and heading off on his quest? Once he finally got going he travelled about 1800 miles, from Bag End to Mount Doom, in about 6 months. It would likely have been quicker by eagle._