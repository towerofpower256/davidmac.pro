---
title: How you should allow elevated users enter and exit the service portal
description: Out-of-the-box ServiceNow isn't great for allowing elevated users to jump in and out the service portal, but it's easily fixed with some small changes.
date: 2020-12-26
tags:
- ServiceNow
- Solution
- Opinion
eleventyExcludeFromCollections: false
---

The self-service Service Portal is a powerful feature of ServiceNow. It provides end users and non-fulfiller users with a clean, user friendly, and straight-forward experience which allows them to easily find what they need to, without overwhelming them with the myriad of navigation options available in the Platform UI (a.k.a. the backend, the ITIL view, the normal view).

![Platform UI vs Service Portal](./platform-ui-sp.jpg)

Here's a screenshot of how the service portal looks as it comes out-of-the-box.

![Baseline service portal screenshot](./ootb-sp-screenshot.jpg)

An issue that I experience regularly is when fulfiller-users want to enter and exit the service portal and aren't sure how, or that power users find it frustrating to frequently enter and exit the service portal by playing with the URL. There's no buttons and no links (except for admins), and that's a problem.

Out of the box, entering and exiting the service portal involves the user has manually updating the URL in the browser address bar, which also requires that the user knows the URL of the Service Portal they want to enter.

![Editing the URL](./editing-address-bar.png)

How can this be made easier to improve the user experience? Here's what I often recommend.

## Design
The design is simple:
* The user will be automatically redirected to the Service Portal if they don't have one of the roles we specify.
* The user can enter the Service Portal from the Application Navigator: **Self-service > Service Portal**.
* The user can exit the Service Portal using a button in the header that only appears if they have one of the roles we specify.

![Design](./design-diagram.svg)

It's at this point that you'll want to decide the user roles that will decide if the user gets redirected to the service portal or not. As it comes out-of-the-box, the **SPEntryPage** script will not redirect any user that has any role of any kind. This works well for simple scenarios, but I find that this is too broad a brush when custom non-fulfiller roles start being used, such as roles to control access to homepages. Whatever works for you.

# Automatically redirecting to the Service Portal
ServiceNow has already made this part easy. Use the script include "SPEntryPage", and modify it slightly so that users with the desired role(s) do not get redirected.

This script is run when the user navigates to the base URL of the instance (if they're not trying to go anywhere). If they have a link to somewhere specific like a record, they won't be redirected, and will successfully land where they were trying to get to.

I think this is a good thing. Redirecting to the service portal typically loses where the user was trying to get to, and users will quickly get frustrated when they don't end up where they wanted to go. This also looks bad if a user clicks on a link to something in an email sent out from ServiceNow and gets mistakenly redirected somewhere else instead.

E.g. 
* instance.service-now.com/ = **redirected**
* instance.service-now.com/incident.do? = **not redirected**
* instance.service-now.com/navto.do?url=incident.do = **not redirected**

> The company "Awesome Inc" only wants IT staff to be able to see the Platform UI. All other staff members should be redirected to the Service Portal.
> HR and Facilities staff should also be redirected.
> 
> If the user has the "itil" role, they will continue to use the Platform UI. Otherwise, they'll be redirected to the Service Portal.

To accommodate this scenario, lets change this line in the **SPEntryPage** script from this:

```js
if (user.hasRoles() && !redirectURL && !isServicePortalURL)
```

to this:

```js
if (user.hasRole("itil") && !redirectURL && !isServicePortalURL)
```

That should be about it. Redirecting users using the **SPEntryPage** script has been extensively covered by a number of authors & documents, here are some links if you'd like to look into the topic further.
* https://hi.service-now.com/kb_view.do?sysparm_article=KB0746730
* https://docs.servicenow.com/bundle/istanbul-servicenow-platform/page/build/service-portal/concept/c_SPSSOLoginAndRedirects.html
* https://serviceportal.io/docs/documentation/sso_configuration.md
	
## Entering the Service Portal from the Platform UI
With the out-of-the-box ServiceNow service portal configuration, the only ways to enter the service portal are:
* To be redirected to it when you login.
* To manually navigate to the service portal by changing the browser address by adding `/sp`.

![Editing the URL](./editing-address-bar.png)

This is a user-experience problem for users that wouldn't normally be redirected. Examples include:
* IT Support staff who want to access the portal to assist an end-user.
* ServiceNow administrators who have to regularly switch in and out of the service portal.
* Knowledge managers who want to check how their knowledge articles appear in the user-facing service portal.

There is a very easy solution for this user-experience problem. Create a new module called **Service portal ➚** in the application navigator under **Self-service**. I like the little "➚" symbol that ServiceNow has been using because it denotes that the link will open in a new tab when you click on it. You shouldn't need to add any role against the module because it'll be visible to everyone, just set:
* **Type** to **URL (from Arguments:)**.
* **Arguments** to `/sp`.
* **Window name** to `_blank` so it opens in a new tab.

![Link to service portal](./service-portal-self-service-link.png)

## Exiting the portal
So you've followed a link into the service portal, but you're an ITIL user and want to get out. How do you do it?

In the out-of-the-box service portal, there's no button or link that you can use to leave. This forces the you to manually change the address in your browser's address bar, chopping off the `/sp?` and everything after it.

A change I recommend is to create an **Exit** menu item on the service portal's header menu. This will appear as a button at the top of the service portal to users with the same roles that don't get redirected, and clicking on it will redirect the user to the platform UI.

To do so, open the Service Portal Menu that's set as the header menu for your service portal and create a new menu item with these details:
* **Name** to `Exit portal`.
* **Order** to `100` or so, to position it on the far-left.
* **Glyph** to the up arrow. I like this because it denotes that you're going to leave the service portal.
* **Type** to `URL`.
* **URL** to just `/`.
* and **Condition** to `gs.hasRole("itil")`. Note that this is the same condition used in the SPEntryPage redirection for consistency.

![Exit portal menu item](./exit-portal-screenshot.png)

And that's it! Fulfiller and power users can now enter and exit the service portal as they like.
I can't recommend this change enough. It can be frustrating for users that get into the portal and have to force their way out of it, more so for users that know that the service portal exists but can't find where it is. These links and buttons cuts down a handful of clicks and key presses down to a single click. This is an easy but worthwhile boost to user experience. 