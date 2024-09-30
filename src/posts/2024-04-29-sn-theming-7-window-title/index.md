---
title: Non-Production Instance Theming Part 7 - Custom Window Title
description: Are you developing in production without realising? Did you accidentally delete that production data, or load test data into your live instance? It's easy to forget which ServiceNow instance you are in. Giving each instance a custom window title can help guide users through a sea of browser tabs.
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
Another way of telling instances apart by the browser tab is to have unique window titles. This approach can be useful if you have many tabs open and you're sifting through them by name.

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

Here's an example of instance-specific "**glide.product.name**" of "**ServiceNow DEV**". You can see how it is visible in the first tab but it is cut-short in the other tabs.

[![Page title custom](page-title-custom.png)](page-title-custom.png)