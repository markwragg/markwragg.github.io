---
title: Synchronising Git repositories to disk with Chef
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2016/06/Chef-Git-Logo-6-1.png"
date: '2016-07-08 12:49:29'
tags:
- chef
---
My team has an ever-growing collection of administration scripts and tools which are organised and maintained within a number of Git repositories. In order to ensure that the copies of these scripts in production always reflect the Master branch in Source Control we set up a Chef cookbook to synchronise the repositories to a network share in the environment. This post details how this functionality was developed and deployed.

**What is Git?**

<div class="image-div" style="width: 300px; float: right; margin: 0px 30px 30px 30px;">
![XKCD Explains Git](http://imgs.xkcd.com/comics/git.png)
<p>Image source: <a href="https://xkcd.com/1597/">xkcd.com</a>
</div>

In case you're not aware, Git is a version / source control system that has been around since 2005 and is widely used by software developers as a way to manage source code. It allows teams to collaborate on code, ensuring that changes that are made can be tracked and ultimately merged in to the "master" branch of the code.

Source Control is typically a relatively new concept for Infrastructure / Operations teams, but there are multiple benefits for working within such a system, not least of which are:

1. Your working behaviour starts to mimic that of your Developer peers, making DevOps collaboration easier and more likely.
2. All changes are tracked and controlled, making it easy to understand what has changed and roll back those changes.
3. It encourages the control of infrastructure as code, which ensures a consistent and repeatable architecture.

*-- If Git is new to you, I recommend you follow [this guide](https://guides.github.com/activities/hello-world/) to get to grips with the basic concepts.*

**What is Chef?**

Chef is an automation platform that allows you to declare infrastructure as code and have that code applied to systems to ensure they are consistent and compliant (to reach a desired system state and to centralise the control of that state).

*-- If Chef is new to you I recommend you check out the [Windows-based getting started guide](https://learn.chef.io/learn-the-basics/windows/configure-a-resource/).*

If any of the above is new to you I also strongly recommend you read [The Release Pipeline Model whitepaper](https://msdn.microsoft.com/en-us/powershell/dsc/whitepapers#the-release-pipeline-model) by Steven Murawski and Michael Greene.

**Why use chef to sync git repos?**

Our environment currently lacks a package management/code deployment solution such as [AppVeyor](http://www.appveyor/com) or [Jenkins](https://jenkins.io/) for continuous integration. But we have started to experiment with Chef for configuration management. We are at various levels of maturity with Chef for our products and we're just starting to look at ways to leverage it for our infrastructure. As a result it was a quick-win to use it for this task until we're ready to implement a more suitable solution.

### How it works
If you'd like to view the source code and/or reuse this cookbook, you can find it in Git here:
> https://github.com/markwragg/Chef-GitSync.

**Usage**

This cookbook is intended to run in a production environment in order to synchronise the master branches of one or more repositories so that when changes are approved and merged they are imminently and automatically promoted to Production. 

However, as you can configure it to sync any branch you define to any location it could have various other uses.

**The recipe**

You need to define the following attributes (in attributes/default.rb):

- Repositories: A comma-separated list of URLs to the .git file for each repository
- Rootpath: The local folder where you want the repositories synced
- Branch: The name of the branch you want synced

For example:

```language-ruby
default['stash-to-disk-sync']['repositories'] = ['https://example.com/example.git','https://example2.com/example2.git']

default['stash-to-disk-sync']['rootpath'] = 'C:\\SourceCode\\'
default['stash-to-disk-sync']['branch'] = 'master'
```

The recipe then simply loops through the list of repositories and calls Git to sync them to a folder named the same as the repository underneath your defined rootpath and syncing the Branch specified:
```language-ruby
node['stash-to-disk-sync']['repositories'].each do |repositoryname|
  foldername = repositoryname.scan(%r{([^\/]+)(?=\.\w+$)}).last.first

  git "#{node['stash-to-disk-sync']['rootpath']}#{foldername}" do
    repository repositoryname
    revision node['stash-to-disk-sync']['branch']
    action :sync
  end
end
```

### Installation

We have implemented this using Chef Zero, which allows us to run it without having a Chef Server installation. The below details how we have set this up in our environment but your setup/needs might vary.

> "Chef Zero is a simple, easy-install, in-memory Chef server that can be useful for Chef Client testing and chef-solo-like tasks that require a full Chef Server. "
>
> Source: https://github.com/chef/chef-zero

**To setup/configure Chef Zero:**

1. Install Chef: https://downloads.chef.io/chef-client/windows/.
2. Create the following files in C:\Chef\ with the contents described below:

C:\Chef\ **client.rb**

```language-ruby
cookbook_path 'C:\\Chef\\Cookbooks\\'
json_attribs 'C:\\Chef\\attributes.json'
chef_zero.enabled true
```

C:\Chef\ **attributes.json**

```language-json
{
	"run_list": [
		"recipe[chef-client::default]",
		"recipe[git::windows]",
		"recipe[git-to-disk::default]"
	],
	"chef_client": {
		"interval": "1800"
	},
	"git-to-disk": {
		"rootpath": "\\\\yourdomain.local\\code\\"
	}
		
}
```
**Download the required Cookbooks:**

You'll now need to download the git-to-disk cookbook and it's dependencies. It requires the git::windows recipe and the chef-client::default recipe, but those each have dependencies on other cookbooks also. As a result the list is quite extensive. 

Download or [Clone](https://git-scm.com/docs/git-clone) each of the following to **C:\Chef\Cookbooks** (or to whatever you have specified as your cookbook_path in client.rb):

1. [Chef-GitSync](https://github.com/markwragg/Chef-GitSync).
- [chef_client](https://github.com/chef-cookbooks/chef-client).
- [Git](https://github.com/chef-cookbooks/git).
- [build-essential](https://github.com/chef-cookbooks/build-essential)
- [chef_handler](https://github.com/chef-cookbooks/chef_handler)
- [cron](https://github.com/chef-cookbooks/cron)
- [dmg](https://github.com/chef-cookbooks/dmg)
- [logrotate](https://github.com/stevendanna/logrotate)
- [seven_zip](https://github.com/daptiv/seven_zip)
- [windows](https://github.com/chef-cookbooks/windows)
- [yum](https://github.com/chef-cookbooks/yum)
- [yum-epel](https://github.com/chef-cookbooks/yum-epel)

**Execute Chef-Client:**

Ensure the directory exists where you want to clone the repositories to (C:\Software\Stash by default). Then to kick off the first run of Chef:

1. Open a **Powershell** or cmd window. 
- Navigate to  to **C:\Chef\\**.
- Execute **Chef-Client**.

![Chef-Client run of Chef-GitSync](/content/images/2016/06/Chef-Client-Git-Sync1.png)

After synchronising the cookbooks, Chef will perform the desired actions. It will check if Chef Client is running as a service, and if it is not will install a service for it. This will then run Chef-Client every 1800 seconds (by default).   

If a Git Client is not already installed, one will be installed. Git is an obvious dependency as it allows it to clone and pull updates from the repositories, but by having Chef install this it means that if someone were to uninstall Git, Chef would reinstall it automatically on its next run - correcting the desired state: 

![](/content/images/2016/06/Chef-Client-Git-Sync2.png)

Finally it will go through each listed repository and use Git to synchronise them to the local folder.

----For my team this is a first small step towards a release pipeline in our environment, which has otherwise been largely governed by business processes. Ultimately this was simple to implement, particularly as we had a pre-existing Source Control to use and because it was possible to access that from our hosting environment. With this in place, we can ensure that any future changes to our automation scripts are only made via Source Control and that those changes are automatically promoted to Production once approved.