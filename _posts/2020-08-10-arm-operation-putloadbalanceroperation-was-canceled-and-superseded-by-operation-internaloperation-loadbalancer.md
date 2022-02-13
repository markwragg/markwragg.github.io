---
title: ARM template error "Operation PutLoadBalancerOperation was canceled and superseded
  by operation InternalOperation"
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2020/08/dropped-icecream-1.jpg"
  teaser: "/content/images/2020/08/dropped-icecream-1.jpg"
date: '2020-08-10 10:02:25'
tags:
- azure
- troubleshooting
- arm
- iac
- infrastructure
---
Last week I spent a day troubleshooting an ARM (Azure Resource Manager) template deployment error that was frustratingly vague. A Load Balancer resource in the template was returning a result of "conflict" and the following error:

> Note: I've blanked out the operation IDs in the example error below as these were unique to my deployment.

```
{
    "status": "Canceled",
    "error": {
        "code": "ResourceDeploymentFailure",
        "message": "The resource operation completed with terminal provisioning state 'Canceled'.",
        "details": [
            {
                "code": "Canceled",
                "message": "Operation was canceled.",
                "details": [
                    {
                        "code": "CanceledAndSupersededDueToAnotherOperation",
                        "message": "Operation PutLoadBalancerOperation (xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx) was canceled and superseded by operation InternalOperation (xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)."
                    }
                ]
            }
        ]
    }
}
```

The template that was being deployed wasn't actually making any modifications to the Load Balancer at all (and was being executed in Incremental deployment mode). However the VMSS (Virtual Machine ScaleSet) associated with the resource was being modified, with changes being made to the extensions configured for the ScaleSet VMs.

The error turned out to be due to a missing dependency. I thought I'd blog it quickly here because there was nothing in the error that really pointed to that as the cause, and other examples I found on Google for this issue also didn't point to dependencies as the issue.

To resolve my issue I just had to make sure that the Load Balancer was listed in the `dependsOn` property for the VMSS it was associated with. Due to some convoluted logic in the template design (which I inherited) copy loops are being used but not all of the items in the array get created by the copy loop, so it had been missed (the existing dependency references the copy loop name). I'm not sure why this had never caught us out before, but the problem only seemed to surface when the VMSS was being modified after the initial deployment, whereas the initial deployment for the template worked just fine.

For more information on how to define resource dependencies in ARM templates see here:

- https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/define-resource-dependency