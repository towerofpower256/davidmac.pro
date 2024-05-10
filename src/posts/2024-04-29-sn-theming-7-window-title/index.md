---
title: 
description: 
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-03-04
tags:
- ServiceNow
- Solution
eleventyExcludeFromCollections: false
series: sn-theming
---

## Instance window title 
Another way of telling instances apart by the browser tab is to have unique window titles.

However, it's impact is minimal as the ends of window titles are usually cut off at the end.

> **Note:** Custom window titles are not visible on the Service Portal.

To quote ServiceNow:

[Modify the banner](https://docs.servicenow.com/bundle/washingtondc-platform-user-interface/page/administer/navigation-and-ui/concept/c_ModifyTheBanner.html)

> *These properties are used to set the window title as follows:*
> 
> *glide.product.name*
> *glide.product.description*
> 
> *If glide.product.name is blank, then the ServiceNow name is used as the product name for the window title.*

Here's how the page titles look out-of-the-box.

[![Page title out of the box](page-title-ootb.png)](page-title-ootb.png)

Here's an example of instance-specific "**glide.product.name**" of "**ServiceNow DEV**". You can see how it is visible in the first tab, but it cut-short in the other tabs.

[![Page title custom](page-title-custom.png)](page-title-custom.png)