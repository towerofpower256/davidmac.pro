---
title: ServiceNow Quick Reference - How Emails are Sent to Groups
description: A quick reference on how ServiceNow sends emails to groups, group members, managers, and group email addresses, and how different settings change how it works.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-02-18
tags:
- ServiceNow
eleventyExcludeFromCollections: false
---

## Group emails
A colleague asked how to get ServiceNow to send notification emails to a group's members as well as the group's email address. So I did some testing to find out how it all works. Hopefully this helps others asking the same question.

Group [sys_user_group] records have 3 fields (sometimes hidden) that we're interested in.
* Group email
* Include members (hidden)
* Exclude manager (hidden)

<table>
    <thead>
        <tr>
            <th>Group has a group email</th>
            <th>Manager is a group member</th>
            <th>Include members</th>
            <th>Exclude manager</th>
            <th></th>
            <th>Sent to group email</th>
            <th>Sent to manager</th>
            <th>Sent to members</th>
            <th>Sent to child groups</th>
        </tr>
    </thead>
    <tbody class="text-light text-center">
        <tr>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td></td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
        </tr>
        <tr>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td></td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
        </tr>
        <tr>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td></td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
        </tr>
        <tr>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td></td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
        </tr>
        <tr>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td></td>
            <td class="bg-success">Yes</td>
            <td class="bg-success">Yes</td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
        </tr>
        <tr>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td></td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
            <td class="bg-dark">No</td>
        </tr>
        <tr>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td class="bg-success">Yes</td>
            <td></td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
            <td class="bg-success">Yes</td>
            <td class="bg-dark">No</td>
        </tr>
    </tbody>
</table>

## Links
* SN Docs - Create a user group
 https://docs.servicenow.com/bundle/washingtondc-platform-administration/page/administer/users-and-groups/task/t_CreateAGroup.html
* SN Docs - Receive notifications (focus on approvals)
 https://docs.servicenow.com/bundle/washingtondc-build-workflows/page/administer/service-administration/concept/c_ReceiveNotifications.html
* SN Community - Group Management (tips)
 https://www.servicenow.com/community/developer-articles/servicenow-group-management/ta-p/2668499
* SN Community - When to use group email field in group record
 https://www.servicenow.com/community/developer-forum/when-to-use-group-email-field-in-group-record/m-p/2251556
* SN Community - How can I send emails to group email mailbox, and not to all of the members of the group?
 https://www.servicenow.com/community/developer-forum/how-can-i-send-emails-to-group-email-mailbox-and-not-to-all-of/m-p/2043110
* SN Community - How ServiceNow builds recipients lists for email notifications
 https://www.servicenow.com/community/now-platform-articles/how-servicenow-builds-recipients-lists-for-email-notifications/ta-p/2317854