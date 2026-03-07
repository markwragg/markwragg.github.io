---
title: The end is nigh! Automate your SSL certificate renewal in Azure
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2026/browser-padlock.webp"
  teaser: "/content/images/2026/browser-padlock.webp"
date: '2026-02-15 12:00:00'
tags:
- azure
- certificates
- automation
- deployment
- arm
draft: true
---

The maximum valid lifetime of SSL certificates is being significantly reduced over the next few years, so there's never been a better time to automate your certificate management. In Azure SSL certificates can be used in lots of different places. Some of which are easier to manage than others, and you may already have a degree of automation in place. But by 2029 SSL certificates will have a maximum lifetime of 47 days, so automation will be essential to ensure you can continue to keep services running without disruption, and without significant overhead on the team or teams that manage and maintain your infrastructure.

In this blog post we'll explore some of the different areas of Azure that utilise SSL certificates and how you might consider automating them.

The reduction of the maximum valid lifetime of public SSL is being implemented in the following phases:

- Until March 14th 2026: Max 398 days (This is/was effectively 13 months, and has been the standard since September 2020)
- From March 15th 2026: Max 200 days
- From March 15th 2027: Max 100 days
- From March 15th 2029: Max 47 days

This means that by 2029 SSL certificates will need to be renewed nearly 8 times per year. What's also worth noting is that if you renew your SSL certificates before the March 14th 2026 deadline (and get the maximum 398 days), there will be no value in renewing those certificates again until October 2026, as you'll get a certificate with a shorter validity than you already have.

 ![The end is nigh cartoon](/content/images/2026/the-end-is-nigh.png){: .align-center}



