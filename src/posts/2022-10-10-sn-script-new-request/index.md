---
title: "How to submit a ServiceNow catalog request using code"
description: Don't manually create ServiceNow catalog request by hand, here's a quick guide on how to use the tools that ServiceNow provides.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2022-10-10
tags:
- ServiceNow
- Coding
eleventyExcludeFromCollections: false
---

Sometimes in ServiceNow, you'll have the need to submit a new Service Catalog Request without the user, typically using script.


It might be:
* A scheduled script job that creates requests when it finds something.
* A shortcut or UI action that should automatically create a new service request from another record.
* A transform map that needs to import data as catalog requests.

There's a lot of ***magic*** that ServiceNow does automatically when you submit a service catalog request. It's important that the way you do it includes this magic.

## Option 1 - sn_sc.CartJS
At the time of writing this article, this is the recommended method to create new RITMs (catalog requests) through script.

This robust script API should do all of the ***magic*** for you that you'd expect when you manually submit a request by hand.

```js
var cart = new sn_sc.CartJS();

var newItem = cart.addToCart({
	"sysparm_id": "SYS_ID_OF_CAT_ITEM",
	"sysparm_quantity": "1",
	"sysparm_requested_for": "SYS_ID_OF_REQUESTED_FOR_USER",
	"variables":{
		"acrobat": "true",
		"photoshop": "false",
		"Additional_software_requirements": ""
	}
});

var checkoutInfo = cart.checkoutCart();
```

There's many more options that you can use when submitting a new request. Check them out here:
https://developer.servicenow.com/dev.do#!/reference/api/tokyo/server/sn_sc-namespace/c_CartJSScoped

## Option 2 - Cart
This is the old **and deprecated** method of creating new RITMs through script. 
Because the script include is deprecated, or you may not be able to view the script at all. You should expect unexpected behaviour when using it.

To see more, open the **script include** "Cart" in your instance of ServiceNow. Once open, you may not be able to view the script because ServiceNow really does not want you to use it.

```js
var id = GlideGuid.generate(null); // Generate a new GUID
var cart = new Cart(id); // Instantiate a fresh cart using the random GUID from earlier
var myRitm = cart.addItem('060f3afa3730001354b6a3545d3e9dbe');  // The sys_id of the catalog item to submit
cart.setVariable(myRitm ,'ram', '16');       /**/
cart.setVariable(myRitm ,'os', 'win10');     /* Catalog item variables can be populated */  
cart.setVariable(myRitm ,'storage', '1tb');  /**/
var request = cart.placeOrder();  // The placeOrder() function returns a GlideRecord for the sc_request record it generates
```

## Option 3 - Create it all yourself
You can always create a new catalog request entirely manually.

***You should not ever do this!*** I'm only doing this to show you a bit of what happens behind the scenes.

In short, ServiceNow does the following when you submit a new request:
* Starts the RITM record.
* Populate the RITM with variables.
* Calculates the price based on the item, chosen options, certain field values, etc.
* Automatically updates the request's "Requested for" from those special "Requested for" variables, if the request has any of them.
* Create the REQ, populating the "Requested for".
 Note that after the REQ's approval state is set to approved, it will automatically update the RITM's underneath it and kick-off their flows / workflows.

```js
// sys_id of the "Standard Laptop" catalog item.
var catItemSysId = "04b7e94b4f7b4200086eeed18110c7fd";

// Some request and request item data that we'll use later.
var variables = {
    "Additional_software_requirements": "I would like this thing",
    "acrobat": "true"
};
var requestDetails = {
    "special_instructions": "Please don't drop the thing",
    "delivery_address": "123 Fake Street"
}

var grCatItem = new GlideRecord("sc_cat_item");
grCatItem.get(catItemSysId);

// Ready the sc_req_item
var grRitm = new GlideRecord("sc_req_item");
grRitm.initialize();
grRitm.cat_item = ""+grCatItem.sys_id;
grRitm.short_description = grCatItem.short_description;

// We need to set and use the sc_req_item's GUID before it gets inserted.
// This method doesn't handle GUID's that already exist, be careful.
var newReqItemGuid = gs.generateGUID(); 
grRitm.setNewGuidValue(newReqItemGuid);

// Don't care about pricing for this example. 
// Normally, that would be calculated dynamically.

// Create the variable instances from cat item variables.
// Variable sets would work in a similar way.
var grCatItemVar = new GlideRecord("item_option_new");
grCatItemVar.addQuery("cat_item", catItemSysId);
grCatItemVar.query();
while (grCatItemVar.next()) {
    var grRitmVar = new GlideRecord("sc_item_option");
    grRitmVar.initialize();
    grRitmVar.item_option_new = ""+grCatItemVar.sys_id;
    grRitmVar.order = ""+grCatItemVar.order;
    if (variables[grCatItemVar.name]) {
        grRitmVar.value = variables[grCatItemVar.name];
    } else {
        grRitmVar.value = ""+grCatItemVar.default_value;
    }
    grRitmVar.insert();

    var grRitmVarM2M = new GlideRecord("sc_item_option_mtom");
    grRitmVarM2M.initialize();
    grRitmVarM2M.request_item = ""+grRitm.sys_id;
    grRitmVarM2M.sc_item_option = ""+grRitmVar.sys_id;
    grRitmVarM2M.insert();
}

// Save the new sc_req_item
grRitm.insert();

// Create the sc_request
var grReq = new GlideRecord("sc_request");
grReq.initialize();
grReq.special_instructions = requestDetails.special_instructions;
grReq.delivery_address = requestDetails.delivery_address;
grReq.insert();

// Update the sc_req_item with the newly create request
grRitm.request = ""+grReq.sys_id;
grRitm.update();

// DONE
// Your new catalog request should be finished and ready to go!
```