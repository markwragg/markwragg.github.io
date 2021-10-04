---
title: Auto Scaling across multiple Availability Zones with CloudFormation
image: "/content/images/2016/05/Autoscaling_MultiAZ_s.png"
date: '2016-05-02 14:00:00'
tags:
- aws
---
A colleague and I recently implemented an improvement to our CloudFormation scripts which enabled the Auto Scaling Group to launch instances across more than one Availability Zone. This blog post documents the changes we made.

If you're not using CloudFormation, its likely a simple case of defining the Availability Zones when you manually create your Auto Scaling Group. As far as I know, you do this at point of creation and cannot change this later (without creating a new ASG) as well as ensuring you have subnets within your VPC for each AZ.

> **Caveats / Lessons learnt**
>
- I am fairly new to AWS and have a basic understanding of CloudFormation. This guidance might not be applicable to every scenario or best practice. We had existing CloudFormation scripts which I simply modified.
- You need to have a VPC Subnet defined for each Availability Zone.
- Reserved Instances are purchased per Availability Zone, so beware if you have existing RIs or wish to use RIs you need to divide your purchases across the AZs you intend to use.
- When you update your CloudFormation stack, your EC2 instances will be rebalanced immediately as a result. This means half of your instances will be terminated and relaunched in the other AZ. If you want to protect yourself from the sudden loss of half your instances, you could temporarily scale up to twice your existing volume first.

### Pre-requisites

As listed above, its first important to ensure that you have a VPC Subnet defined per Availability Zone you wish to use. This is a simple case of navigating to VPC > Subnets and creating a new Subnet within your VPC that is associated with the new AZ you wish to use. You should likely also ensure that this subnet is associated with the same Route Table as the existing subnet used by the Auto Scaling Group (as defined under VPC > Route Tables > Subnet Associations. If you just use a default main Route Table this may occur by default.

### Modifying the CloudFormation script

Previously our CloudFormation had a single "AvailabilityZone" node under "Parameters". We duplicated this and renamed one to be "AvailabilityZoneA" and the other "AvailabilityZoneB". Previously this was a String type, but we took the opportunity at the same time to update the Type to be: "AWS::EC2::AvailabilityZone::Name". This makes CloudFormation prompt you with a drop down list of the AZs when you are launching or configuring the stack.

```language-json
    "AvailabilityZoneA": {
		"Type": "AWS::EC2::AvailabilityZone::Name",
		"Default": "eu-west-1a",
		"Description": "Name of the first availability zone in which the servers will be created."
    },
  	  
     "AvailabilityZoneB": {
		"Type": "AWS::EC2::AvailabilityZone::Name",
		"Default": "eu-west-1b",
		"Description": "Name of the second availability zone in which the servers will be created."
    },
```
We next made the same modification to the VPCSubnet parameter, replacing the single parameter with two:
```language-json
    "VPCSubnetA": {
	  	"Type": "AWS::EC2::Subnet::Id",
	  	"Description": "The ID of the subnet in the first AvailabilityZone as specified in AvailabilityZoneA."
    },
	  
    "VPCSubnetB": {
	  	"Type": "AWS::EC2::Subnet::Id",
	  	"Description": "The ID of the subnet in the second AvailabilityZone as specified in AvailabilityZoneB."
    },
```
	  
In the resources section we modified the "WebServerGroup" such that the AvailabililityZones and VPCZoneIdentifier properties referenced the values of newly defined parameters.
```language-json
    "WebServerGroup": {
			"Type": "AWS::AutoScaling::AutoScalingGroup",
			"Properties": {
			  "AvailabilityZones" : [{ "Ref" : "AvailabilityZoneA"}, { "Ref" : "AvailabilityZoneB"}],
			 ...	
			 "VPCZoneIdentifier":[{ "Ref" : "VPCSubnetA" },{ "Ref" : "VPCSubnetB" }]
			}
    },
```
We modified the "ElasticLoadBalancer" resource to also reference the two VPC parameters:
```language-json
    "ElasticLoadBalancer": {
			"Type": "AWS::ElasticLoadBalancing::LoadBalancer",
			"Properties": {
				...
				"Subnets" : [{ "Ref" : "VPCSubnetA" },{ "Ref" : "VPCSubnetB" }],
    },
```
We made one final optional change, which was to include the following property in the ElasticLoadBalancer configuration:
```language-json
    "CrossZone": "True",
```
This setting is described here in the AWS documentation: http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/enable-disable-crosszone-lb.html but the description is somewhat difficult to interpret. My interpretation is that this ensures when your ELB determines which Instance to send traffic to, it is considering the load of all instances in all AZs. I believe without this being enabled, it will first pick an AZ (assumedly in a round-robin fashion) and then decide where to send traffic within that AZ based on load. 

### Pushing to production

After committing the .json file changes to Source Control, we scheduled the change and via CloudFormation performed an "Update Stack" operation to bring the changes in to Production. As noted earlier, in each region this caused half of the instances to be immediately terminated and relaunched. After we had confirmed that they were each marked as "InService" (healthy) in the LoadBalancer, we terminated the other pre-existing instances to ensure the ASG was fully working as well as to relaunch them with the latest AMI. The final result was the instances spread equally across two Availability Zones in each of our regions.