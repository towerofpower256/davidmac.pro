---
title: "ServiceNow Script: GlideRecord to JSON"
description: Get a plain JSON object from a ServiceNow record without hard-coding. Perfect for integrations!
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2021-06-10
tags:
- ServiceNow
- Coding
eleventyExcludeFromCollections: false
---

## My script
Sometimes, you want to get a record from ServiceNow as a simple Javascript object.

This is a pretty popular thing to do, especially for integrations where you're passing data around as JSON payloads.

Don't hard-code those mappings, building your object 1 field at a time, just throw it through this function and let the magic do its work!

```js
// This will get a GlideRecord as a mostly flat(ish) object.
// Intended to get a GlideRecord's details that are ready to be turned into a JSON message.

// gr = The GlideRecord to work on.
// fields = a string array of fields to include in the object from the glide record.
// Returns an object, ready to be JSON-ified.

// Usage example: 
// var obj = getGrObject(grIncident, ["sys_id", "caller_id", "description"]);
// var jsonObj = JSON.stringify(obj);
// {
//   "sys_id": {
//     "value": "1c741bd70b2322007518478d83673af3",
//     "display_value": ""
//   },
//   "caller_id": {
//     "value": "681ccaf9c0a8016400b98a06818d57c7",
//     "display_value": "Joe Employee"
//   },
//   "description": {
//     "value": "I am unable to connect to the email server. It appears to be down.",
//     "display_value": ""
//   }
// }


function getGrObject (gr, fields) {	
	var obj = {};

	// If a list of fields has not been provided, use all fields
	if (!fields) {
		fields = [];
		// getElements returns a Java array. Gotta use .size() and .get()
		var elements = gr.getElements(); 
		for (var i=0; i < elements.size(); i++)
			fields.push(elements.get(i).getName());
	}

	for (var iField=0; iField < fields.length; iField++) {
		var fname = fields[iField];
		if (!gr.isValidField(fname)) {
			addElement(obj, fname, "", ""); // field doesn't exist, just add blank
			continue;
		} 

		var ed = gr.getElement(fname).getED(); // Get the Element Descriptor for this field
		var isChoiceField = ed.isChoiceTable(); // Check if this field is a choice field
		var fType = ""+ed.getInternalType(); // Get the field's type name

		// Choice field
		if (isChoiceField) {
			// There's a special function to get the display value of a choice field
			addElement(obj, fname, gr.getValue(fname), gr.getElement(fname).getChoiceValue());
			continue;
		}
		
		// Boolean
		if (fType == "boolean") {
			// Raw boolean values are either a 0 (false) or a 1 (true)
			var boolValue = gr.getValue(fname) == 1;
			addElement(obj, fname, boolValue, "");
			continue;
		}

		// Fields that should return a value and a display value
		var displayValueFields = ["reference", "glide_date", "glide_time", "glide_date_time"];
		if (displayValueFields.indexOf(fType) > -1) {
			addElement(obj, fname, gr.getValue(fname), gr.getDisplayValue(fname));
			continue;
		}

		// Ordinary fields
		addElement(obj, fname, gr.getValue(fname), "");
	}

	// Return the result
	return obj;

	function addElement(obj, fname, value, displayValue) {
		obj[fname] = {};
		obj[fname]["value"] = value || "";
		obj[fname]["display_value"] = displayValue || "";
	}
}
```

## GlideQuery
New in the Paris release of ServiceNow is a new class called **GlideQuery**. [Andrew Albury-Dor](https://andrew.alburydor.com/) let me know about this one, and how you can specify the fields that you want to return.

https://developer.servicenow.com/dev.do#!/reference/api/paris/server/no-namespace/GlideQueryAPI#GQ-get_S_O?navFilter=glidequery

If you want to get the display value of a field, you can add `$DISPLAY` to the end of a field name.

E.g. `caller_id$DISPLAY`

However, it's worth noting that it doesn't allow you to dot-walk through reference fields to get values. 

E.g. you can't use it to get the manager of an incident's assignment group.

```js
var gqIncident = new global.GlideQuery("incident")
  .get('552c48888c033300964f4932b03eb092', ["number", "caller_id", "caller_id$DISPLAY", "caller_id.name", "caller_id.email"]);

gs.info(JSON.stringify(gqIncident));
```

```json
{
	"_value": {
		"sys_id": "552c48888c033300964f4932b03eb092",
		"number": "INC0010112",
		"caller_id": "8d56406a0a0a0a6b004070b354aada28",
		"caller_id$DISPLAY": "Eric Schroeder"
	},
	"_lazyValueFetched": false
}
```

## GlideSPScriptable
I'm not 100% comfortable with using GlideSPScriptable outside of the Service Portal, however it does the job of JSON-ing GlideRecords if you need to.

### getFieldsObject
If you've worked with the ServiceNow Service Portal before, you've likely seen this line of code in a number of widgets:
```js
$sp.getFieldsObject(gr, fields);
```

This does something similar to my script above, and returns a plain object with the information about the fields.

```js
var grIncident = new GlideRecord("incident");
grIncident.query(); grIncident.next();
gs.print(JSON.stringify(new GlideSPScriptable().getFieldsObject(
	grIncident, 
	"sys_id,caller_id,assignment_group,assigned_to,short_description,description"
	)));
```

```json
{
	"sys_id": {
		"name": "sys_id",
		"label": "Sys ID",
		"value": "1c741bd70b2322007518478d83673af3",
		"display_value": "1c741bd70b2322007518478d83673af3",
		"type": "GUID"
	},
	"caller_id": {
		"name": "caller_id",
		"label": "Caller",
		"value": "681ccaf9c0a8016400b98a06818d57c7",
		"display_value": "Joe Employee",
		"type": "reference"
	},
	// -- snip --
}
```

### getForm
Similar to the above, you've probably seen this line being used in Service Portal widgets:
```js
$sp.getForm("incident", sys_id);
```

This result is a big object relevant to a form, and more. It's worth noting that this function returns more than just the values of a record, but all of the information to render a form. This bulk means calling this function can take as long as it would to open the form page for this record, which is much slower than other methods. 

It includes information relative to a form, including:
* Form layout
* Fields and field metadata
* Table metadata
* UI actions
* UI scripts
* Related lists
* Client scripts
* Data policies

I can't imagine that you want all of this information, unless you were actually looking to render a full form. I wouldn't recommend using `getForm` if all you want is the information.

```js
var grIncident = new GlideRecord("incident");
grIncident.query(); grIncident.next();
gs.print(JSON.stringify(
		new GlideSPScriptable().getForm("incident", grIncident.sys_id)
	));
```

*I'm not going to provide a sample because the result is rather large. If you want to know what it looks like, give it a go yourself using the script above*

## Running JSON.stringify a GlideRecord object
*But David, can't I just use `JSON.stringify` directly on a GlideRecord?*

Sadly no, doing `JSON.stringify` directly on a GlideRecord object doesn't work the way that you'd like. Because all of the elements in the GlideRecord object are GlideElement objects instead of strings, the stringifier doesn't really like that.

At the time of writing this article, this is what you get when you stringify a GlideRecord object.

```js
var gr = new GlideRecord("incident");
gr.query(); gr.next();
gs.print(JSON.stringify(gr));
```

```json
{
  "parent": {},
  "caused_by": {},
  "watch_list": {},
  "upon_reject": {},
  "sys_updated_on": {},
  "approval_history": {},
  "skills": {},
  "number": {},
  "u_all_approvers": {},
  "state": {},
  "sys_created_by": {},
  "knowledge": {},
  "order": {},
  "delivery_plan": {},
  "cmdb_ci": {},
  "impact": {},
  "contract": {},
  "active": {},
  "work_notes_list": {},
  "priority": {},
  "sys_domain_path": {},
  "rejection_goto": {},
  "group_list": {},
  "business_duration": {},
  "approval_set": {},
  "wf_activity": {},
  "needs_attention": {},
  "universal_request": {},
  "short_description": {},
  "correlation_display": {},
  // and so on...

  "sys_meta": {
    "active": "1",
    "array": "0",
    "attributes": "all_tables.query_hints=true,email_client=true,hasWorkflow=true,live_feed=true",
    "audit": "1",
    "calculation": "",
    "choice": "1",
    "choice_field": "",
    "choice_table": "",
    "create_roles": "",
    "default_value": "",
    "delete_roles": "itil_admin",
    "dependent": "",
    "dependent_on_field": "",
    "display": "number",
    "dynamic_creation": "0",
    "dynamic_creation_script": "",
    "dynamic_default_value": "",
    "dynamic_ref_qual": "",
    "element_reference": "0",
    "filterable": "1",
    "foreign_database": "",
    "function_definition": "",
    "function_field": "0",
    "groupable": "1",
    "help": "",
    "hint": "",
    "i18n_sortable": "1",
    "internal_type": "collection",
    "label": "Incident",
    "language": "en",
    "mandatory": "0",
    "matchable": "1",
    "max_length": "40",
    "multi_text": "0",
    "name": "incident",
    "plural": "Incidents",
    "primary": "0",
    "read_only": "0",
    "read_roles": "",
    "reference": "",
    "reference_cascade_rule": "",
    "reference_floats": "0",
    "reference_key": "",
    "reference_qual": "",
    "reference_qual_condition": "",
    "reference_type": "",
    "sizeclass": "74",
    "sortable": "1",
    "spell_check": "0",
    "staged": "0",
    "sys_package": "ed21bdaa2ff300107087b2e72799b6fa",
    "sys_scope": "global",
    "table_reference": "0",
    "text_index": "1",
    "type": "0",
    "type_description": "collection",
    "unique": "0",
    "url": "",
    "url_target": "",
    "use_dynamic_default": "0",
    "use_reference_qualifier": "simple",
    "virtual": "0",
    "widget": "",
    "write_roles": "",
    "xml_view": "0"
  }
}
```