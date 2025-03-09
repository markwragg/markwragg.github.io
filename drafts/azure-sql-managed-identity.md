---
title: Azure SQL Managed Identity
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/azure-sql.jpg"
  teaser: "/content/images/2025/azure-sql.jpg"
date: "2025-03-09 09:00:00"
tags:
  - azure
  - sql
  - azuredevops
  - powershell
---

I was recently tasked with enabling connectivity to an Azure SQL database via the managed identity of a Logic App and it was surprisingly complicated to setup. In addition we wanted this configuration to be enabled through our automation pipelines, which added a further complexity.

From the official documentation [this guide on configuring Microsoft Entra authentication with Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure) is probably the best place to start. The page on [Microsoft Entra service principals with Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal?view=azuresql) was also useful.


https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure
https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal
https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal-tutorial
https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-directory-readers-role
https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-directory-readers-role-tutorial
https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments-cli
https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/groups-concept
https://learn.microsoft.com/en-us/graph/migrate-azure-ad-graph-configure-permissions

https://learn.microsoft.com/en-us/cli/azure/ad/group/member
https://learn.microsoft.com/en-us/cli/azure/role/assignment
https://learn.microsoft.com/en-us/cli/azure/sql

https://stackoverflow.com/questions/55637169/invoke-sqlcmd-with-aad-authentication