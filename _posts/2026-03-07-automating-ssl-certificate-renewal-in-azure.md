---
title: Automating SSL/TLS certificate renewal in Azure
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2026/internet.jpg"
  teaser: "/content/images/2026/internet.jpg"
date: '2026-03-07 12:00:00'
tags:
- azure
- certificates
- automation
- deployment
- arm
draft: true
---

This blog post is part of the [Azure Spring Clean 2026](https://www.azurespringclean.com/) virtual community event, promoting well-managed Azure tenants.

The end of (annual SSL/TLS Certificate renewals) is nigh! The maximum validity of SSL/TLS certificates is being [significantly reduced over the next few years](https://www.digicert.com/blog/tls-certificate-lifetimes-will-officially-reduce-to-47-days), so there's never been a better time to automate your certificate management. In Azure TLS certificates can be used in lots of different places, some of which are easier to manage than others, and for some services you may already have a degree of automation in place. But by 2029 TLS certificates will have a maximum lifetime of 47 days, so automation will be essential to ensure you can continue to keep services running without disruption and significant overhead on the team (or teams) that maintain your infrastructure.

In this blog post we'll explore some of the different areas of Azure that utilise TLS certificates and how you might consider automating them.

The reduction of the maximum valid lifetime of TLS certificates is being implemented in the following phases:

- **Until March 14th 2026:** Max 398 days (This is/was effectively 13 months, and has been the standard since September 2020)
- **From March 15th 2026:** Max 200 days (6 months + 1/2 of a 30-day month + 1 day)
- **From March 15th 2027:** Max 100 days (3 months + 1/4 of a 30-day month + 1 day)
- **From March 15th 2029:** Max 47 days (31 days + 1/2 of a 30-day month + 1 day)

This means that by 2029 TLS certificates will need to be renewed nearly _8 times per year_. What's also worth noting is that if you renew your TLS certificates before the March 14th 2026 deadline (and get the maximum 398 days), there will be no value in renewing those certificates again until after October 2026, as you'll otherwise get a certificate with a shorter validity than you already have.

If this isn't something you've thought much about up to now, a good first step would be to document everywhere in your infrastructure that currently uses TLS certificates. Some of the resource types to consider include:

- App Service
- Application Gateway
- Front Door
- AKS Ingress
- API Management
- Virtual Machines (VMs) or ScaleSets (VMSS) running webserver technologies such as IIS, NGINX or Apache Tomcat
- Service Fabric

> Note: This is not an exhaustive list.

Automating the generation of your certificates will depend on your certificate provider. There is a standard protocol for certificate renewal automation called [ACME](https://en.wikipedia.org/wiki/Automatic_Certificate_Management_Environment) (Automated Certificate Management Environment) and a number of tools that can be implemented to automate the process with a variety of providers (including Lets Encrypt, which provides free public TLS certificates). Tools include:

- [Posh-ACME](https://poshac.me/docs/v4/): A PowerShell module and ACME client to create publicly trusted TLS/TLS certificates from an ACME capable certificate authority such as Let's Encrypt.
- [win-acme](https://www.win-acme.com/): A Windows-focused ACME client (WACS) that can be scripted to manage certificate renewal, particularly for IIS and Azure-hosted workloads.
- [az-acme](https://azacme.dev/)): A specialized CLI designed to integrate ACME issuers (like Let's Encrypt) directly with Azure Key Vault, storing certificates and enabling auto-rotation.
- [EZCA](https://www.keytos.io/azure-pki): A dedicated Azure-native tool for automating certificate renewal across various Azure services, including support for Key Vault auto-rotation.
- [Certbot](https://certbot.eff.org/): While general-purpose, Certbot can be used with hook scripts to renew certificates and push them to Azure resources.

These tools not only automate the request and retrieval of a certificate, but can automate the process of domain validation, such as adding a token value as a TXT DNS entry to the domain to prove ownership. They can also automate some of the deployment and configuration scenarios, such as updating the certificate binding in IIS. Most of these tools are free and open source, so well worth consideration before you inadvertently reinvent the wheel.

Another tool worth considering is [haveibeenexpired.com](https://www.haveibeenexpired.com/) which you can use to monitor your publicly accessible domains for expiring certificates. When you automate a process, you should also monitor it (trust, but verify).

### Azure Key Vault

Azure Key Vault is the recommended resource to use in Azure for provisioning, managing and deploying TLS certificates. Many other resources in Azure are designed to consume certificates directly from Key Vault, and a single Key Vault can be used to provide certificates to multiple resources.

There are a few ways to [get your certificates in to Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/certificates/certificate-scenarios):

1. Import a TLS certificate you've purchased from an external provider
2. Generate a CSR (Certificate Signing Request) via Azure Key Vault which you can then use to complete a certificate request via an external provider
3. Use one of the partnered CA providers (Digicert or GlobalSign) to generate and import the certificate directly. This requires having an account with either Digicert or Globalsign and providing your credentials for these within the Key Vault.

When you upload a certificate into Key Vault, it imports various properties including the valid from and valid to dates, which you can then use to monitor the validity of your certificates (such as by configuring alerts in Azure Monitor). When a certificate is generated by Key Vault, you configure an email notification to be sent when a certain percentage of the certificates lifetime has been consumed. You can setup the same for imported certificates, by configuring "Certificate Contacts" on the Key Vault, and going to the Issuance Policy settings of the certificate to configure the percentage you want to be notified at. By default this is 80%.

When you use one of the partnered CA (Certificate Authority) providers, Key Vault refers to this as using an [Integrated CA](https://learn.microsoft.com/en-us/azure/key-vault/certificates/how-to-integrate-certificate-authority) and you can configure auto renewal.

You can deploy a Key Vault via Bicep as follows:

```powershell
param location string = resourceGroup().location
param keyVaultName string = 'kv-example'

resource keyVault 'Microsoft.KeyVault/vaults@2023-02-01' = {
  name: keyVaultName
  location: location
  properties: {
    tenantId: subscription().tenantId
    sku: {
      family: 'A'
      name: 'standard'
    }
    enableRbacAuthorization: true
  }
}
```

### App Service

Azure App Service allows you to create a [free managed TLS certificate](https://learn.microsoft.com/en-us/azure/app-service/configure-ssl-certificate) (provided by Digicert), which is then fully managed by the App Service and automatically renewed prior to expiry. If you instead provide your own TLS certificate, you can either upload this directly into App Service, or import it from a Key Vault. The latter is recommended, as its then easier (per the above) to monitor and manage your certificate. When you update the certificate entry in Key Vault, App Service automatically syncs the new version with 24 hours and without downtime. You can also trigger a manual sync within App Service.

> Note when targetting the certificate in Key Vault you must use the non-version specific URL. Doing so will ensure it always pulls the latest certificate.

If you upload the certificate directly to App Service, you will need to modify the certificate in the App Service config, either via the Portal, or via infrastructure code such as ARM or Bicep. Managing the certificate in Key Vault decouples the management of the certificate from the configuration of the resource.

Here's an example of how you might configure an App Service to use a Key Vault certificate via Bicep:

```powershell
param location string = resourceGroup().location
param appName string = 'myapp-${uniqueString(resourceGroup().id)}'
param kvName string = 'kv-example'
param certName string = 'mySslCert'
param customHostname string = 'www.mycustomdomain.com'

resource appServicePlan 'Microsoft.Web/serverfarms@2023-01-01' = {
  name: 'asp-${appName}'
  location: location
  sku: {
    name: 'P1v2'
    tier: 'PremiumV2'
  }
}

resource kv 'Microsoft.KeyVault/vaults@2023-02-01' existing = {
  name: kvName
}

resource webApp 'Microsoft.Web/sites@2023-01-01' = {
  name: appName
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true
  }
}

resource certificate 'Microsoft.Web/certificates@2023-01-01' = {
  name: certName
  location: location
  properties: {
    keyVaultId: kv.id
    keyVaultSecretName: certName
    serverFarmId: appServicePlan.id
  }
}
```

If you reference the certificate from Key Vault, you need to grant the App Service permission to read the certificate via the `Key Vault Secrets User` role. The simplest way to grant this is to configure the App Service to have a System Assigned Identity and then grant this Identity this permission on the Key Vault.

### Application Gateway

Similar to App Service, you can implement a TLS certificate in Application Gateway by either configuring it directly on the resource, or via a Key Vault secret. As before, I recommend using a Key Vault to decouple management of the certificate from configuration of the resource.

Here's a partial example of how you might configure an Application Gateway to use a certificate from a Key Vault via Bicep:

```powershell
param location string = resourceGroup().location
param appGwName string = 'agw-example'
param keyVaultName string = 'kv-example'
param certSecretName string = 'appgw-cert'
param vnetName string = 'agw-vnet'
param subnetName string = 'agw-subnet'

resource keyVault 'Microsoft.KeyVault/vaults@2023-02-01' existing = {
  name: keyVaultName
}

resource appGateway 'Microsoft.Network/applicationGateways@2023-05-01' = {
  name: appGwName
  location: location
  identity: {
    type: 'UserAssigned'
  }

  properties: {
    sku: {
      name: 'Standard_v2'
      tier: 'Standard_v2'
      capacity: 2
    }

    gatewayIPConfigurations: [
      {
        name: 'gwIpConfig'
        properties: {
          subnet: {
            id: resourceId('Microsoft.Network/virtualNetworks/subnets', vnetName, subnetName)
          }
        }
      }
    ]

    frontendIPConfigurations: [
      {
        name: 'frontendIp'
        properties: {
          publicIPAddress: {
            id: resourceId('Microsoft.Network/publicIPAddresses', 'agw-pip')
          }
        }
      }
    ]

    frontendPorts: [
      {
        name: 'httpsPort'
        properties: {
          port: 443
        }
      }
    ]

    sslCertificates: [
      {
        name: 'sslCertFromKV'
        properties: {
          keyVaultSecretId: 'https://${kvName}.vault.azure.net/secrets/${certSecretName}'
        }
      }
    ]

    httpListeners: [
      ..
    ]
  }
}
```

### Front Door



### AKS Ingress



### API Management



### Virtual Machines or ScaleSets



### Service Fabric



## Summary

If you're reading this in 2026, there's still a good amount of time to get a handle on your certificate management before the activity becomes a monthly task. When considering how to manage certificate in Azure, I recommend the following:

- Audit your estate and ensure you have a good understanding of everywhere certificates are used
- Use the fully managed certificate services where possible
- Centralise certificate management via Key Vault, and try and minimise the number of Key Vaults used to store certificates
- Implement monitoring of certificates to ensure you catch expiring certificates before they become an outage