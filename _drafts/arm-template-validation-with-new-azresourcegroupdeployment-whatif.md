---
title: Validating ARM Templates with New-AzResourceGroupDeployment -WhatIf
header:
  overlay_image: "/content/images/2020/08/Grumpy-Cat.jpg"
---
Microsoft have recently released a `WhatIf` operation feature for ARM (Azure Resource Manager) deployments. The feature is still in "preview" but its now freely available for anyone to try. While the results of the tool aren't perfect, its a game changer for developing ARM templates and introduces a Terraform-like preview of changes without needing to maintain a local state file.

Previously the tools for validating ARM templates were pretty limited. There was a basic command for ensuring the syntax of your template was correct, but beyond that having any foresight into what changes a deployment would trigger was pretty limited. In general the best approach was to have a non-Production environment that you could test your deployments against first to ensure you got the result you expected. Even so, it could be easy to misconfigure your environment and have those changes still be "valid" from a deployment perspective. Equally it was easily possible to have a deployment fail for some small reason that then wasn't caught until deployment time. With the `WhatIf` feature catching those kind of issues is quicker and easier.

 