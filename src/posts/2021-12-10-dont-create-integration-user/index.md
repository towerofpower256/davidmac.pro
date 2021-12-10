---
title: Please never create an integration user called "integration.user"
description: This is a really bad idea. Plese don't ever create an integration user called "integration.user".
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2021-12-10
tags:
- ServiceNow
- Coding
- Integration
- Opinion
eleventyExcludeFromCollections: false
---

Please don't create an integration service user account called "**integration.user**". ServiceNow user, AD user, SAP user, it doesn't matter which system, please don't do that.

Please don't use a single user called "integration.user" across multiple different integrations.

Just don't. Please don't.

What integration is it for?

If I disable this account, have I just disabled lots of other integrations?

If I have to give extra powers and security rights to "integration.user" because something needs it, does that mean that every integration gains that extra access?

If I have to change the password for "integration.user", how many different teams should I notify?

When something goes wrong and the logs say the "integration.user" caused it, who or what integration should I be angry at?

It's just a really bad idea, please don't ever do this.