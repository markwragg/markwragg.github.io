---
title: Automating SSL certificate renewal in Azure
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2026/padlock-fence.jpg"
  teaser: "/content/images/2026/padlock-fence.jpg"
date: '2026-02-15 12:00:00'
tags:
- azure
- certificates
- automation
- deployment
- arm
draft: true
---

The maximum lifetime of certificates is being significantly reduced over the next few years, so there's never been a better time to automate your certificate management.

The maximum lifetime of an SSL certificate is being reduced in the following phases:

- Until March 14th 2026: Max 398 days (13 months)
- From March 15th 2026: Max 200 days
- From March 15th 2027: Max 100 days
- From March 15th 2029: Max 47 days

This means that by 2029 SSL certificates will need to be renewed nearly 8 times per year.

 ![The end is nigh cartoon](/content/images/2026/the-end-is-nigh.png){: .align-center}