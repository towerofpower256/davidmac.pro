---
title: Non-Production Instance Theming Part 2 - Custom Product Name
description: Are you developing in production without realising? Did you accidentally delete that production data, or load test data into your live instance? It's easy to forget which ServiceNow instance you are in. Giving all of your ServiceNow instances their own custom product name and banner text can be a constant reminder of which instance you are in.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-03-04
tags:
- ServiceNow
- Solution
eleventyExcludeFromCollections: false
series: sn-theming
---

## Product description text
[![Next Experience logo, hover to get the product description](banner-logo-polaris-hover.png)](banner-logo-polaris-hover.png)

You can change this text to give it some flair.

[![Next Experience logo, hover to get the product description, with custom text](banner-logo-polaris-hover-custom.png)](banner-logo-polaris-hover-custom.png)

To change the "Product Description".
1. Navigate to "**System Properties &gt; Basic Configuration**".
1. Look for and update the "**Page header caption**" [glide.product.description] field.
1. Click on "**Save**".

These changes are considered theming and will survive a clone over the instance.

[![Config page for configuring the product description](screenshot-system-config-logo.png)](screenshot-system-config-logo.png)

> As of the Utah release in the Next Experience UI, ServiceNow recommends not changing the "Product description" [glide.product.description] and instead using Banner Announcements with "Non-dismissible" enabled so it's always there.

> This change will not affect the Service Portal which uses its own configuration for the logo and product name.
> [![Service Portal product name hover](banner-logo-sp-hover.png)](banner-logo-sp-hover.png)