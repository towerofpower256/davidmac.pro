---
title: How the user's status affects ServiceNow emails
description: A quick reference on how the user's status in ServiceNow affects emails being sent and received.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2023-10-19
tags:
- ServiceNow
eleventyExcludeFromCollections: false
---

## Sending emails
Email notifications from ServiceNow may not be sent out to users depending on the user's status.

| Active | Locked | Email sent to user |
| --- | --- | --- |
| Active | Not locked | <span class="text-success fw-bold">Sent</span> |
| Active | Locked out | <span class="text-success fw-bold">Sent</span> |
| Inactive | Not locked | <span class="text-danger fw-bold">Not sent</span> |
| Inactive | Locked out | <span class="text-danger fw-bold">Not sent</span> |

## Receiving emails
Incoming emails may be ignored and disregarded by ServiceNow depending on the user's status.

| Active | Locked | Email sent to user |
| --- | --- | --- |
| Active | Not locked | <span class="text-danger fw-bold">Received, but not processed</span> |
| Active | Locked out | <span class="text-success fw-bold">Received and processed</span> |
| Inactive | Not locked | <span class="text-danger fw-bold">Received, but not processed</span> |
| Inactive | Locked out | <span class="text-danger fw-bold">Received, but not processed</span> |