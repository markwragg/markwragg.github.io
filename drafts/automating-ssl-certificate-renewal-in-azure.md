---
title: Automating SSL certificate renewal in Azure
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2026/automate-cert-renewal.png"
  teaser: "/content/images/2026/automate-cert-renewal.png"
date: '2026-03-07 12:00:00'
tags:
- azure
- certificates
- automation
- deployment
- arm
draft: true
---

The maximum valid lifetime of SSL certificates is being significantly reduced over the next few years, so there's never been a better time to automate your certificate management. In Azure SSL certificates can be used in lots of different places, some of which are easier to manage than others, and you may already have a degree of automation in place. But by 2029 SSL certificates will have a maximum lifetime of 47 days, so automation will be essential to ensure you can continue to keep services running without disruption, and without significant overhead on the team or teams that manage and maintain your infrastructure.

In this blog post we'll explore some of the different areas of Azure that utilise SSL certificates and how you might consider automating them.

The reduction of the maximum valid lifetime of public SSL is being implemented in the following phases:

-*Until March 14th 2026:* Max 398 days (This is/was effectively 13 months, and has been the standard since September 2020)
- *From March 15th 2026:* Max 200 days
- *From March 15th 2027:* Max 100 days
- *From March 15th 2029:* Max 47 days

This means that by 2029 SSL certificates will need to be renewed nearly _8 times per year_. What's also worth noting is that if you renew your SSL certificates before the March 14th 2026 deadline (and get the maximum 398 days), there will be no value in renewing those certificates again until after October 2026, as you'll otherwise get a certificate with a shorter validity than you already have.

If this isn't something you've thought much about up to now, a good first step would be to document everywhere in your infrastructure that currently use SSL certificates. Some of the resource types to consider include:

- App Service
- Application Gateway
- Front Door
- AKS Ingress
- API Management
- Virtual Machines (VMs) or ScaleSets (VMSS) running webserver technologies such as IIS, NGINX or Apache Tomcat
- Service Fabric

## Azure Key Vault

Azure Key Vault is the recommended resource to use in Azure for provisioning, managing and deploying SSL certificates. Many other resources in Azure are designed to consume certificates directly from Key Vault.

There are a few ways to get your certificates in to Key Vault:

1. Import an SSL certificate you've generated via an external provider
2. Generate a CSR (Certificate Signing Request) via Azure Key Vault which you can then use to complete a certificate request via an external provider
3. Use one of the partnered CA providers to generate and import the certificate directly. This requires having an account with either Digicert or Globalsign and providing your credentials for these within the Key Vault.

Automating the generation of your certificates will depend on your certificate provider. There is a standard protocol for certificate renewal automation called ACME (Automated Certificate Management Environment) and a number of tools that can be implemented to automate the process with a variety of providers (including Lets Encrypt, which provides free public SSL certificates). Tools include:

- [Posh-ACME](https://poshac.me/docs/v4/): A PowerShell module and ACME client to create publicly trusted SSL/TLS certificates from an ACME capable certificate authority such as Let's Encrypt.
- [win-acme](https://www.win-acme.com/): A Windows-focused ACME client (WACS) that can be scripted to manage certificate renewal, particularly for IIS and Azure-hosted workloads.
- [az-acme](https://azacme.dev/)): A specialized CLI designed to integrate ACME issuers (like Let's Encrypt) directly with Azure Key Vault, storing certificates and enabling auto-rotation.
- [EZCA](https://www.keytos.io/azure-pki): A dedicated Azure-native tool for automating certificate renewal across various Azure services, including support for Key Vault auto-rotation.
- [Certbot](https://certbot.eff.org/): While general-purpose, Certbot can be used with hook scripts to renew certificates and push them to Azure resources.

## App Service

Azure App Service

## Application Gateway



## Front Door



## AKS Ingress



## API Management



## Virtual Machines or ScaleSets



## Service Fabric