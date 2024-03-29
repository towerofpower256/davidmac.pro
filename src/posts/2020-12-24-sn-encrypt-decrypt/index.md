---
title: Encrypting and decrypting data in ServiceNow
description: Passwords and credentials can be encrypted in ServiceNow. Here's how you can decrypt the information to use in your own customisations and integrations.
date: 2020-12-24
tags:
- ServiceNow
- Coding
eleventyExcludeFromCollections: false
---

In ServiceNow, there are 2 different kinds of password field types for you to choose.

**Password (1 Way Encrypted)**
These are 1-way encrypted fields. Once the data is encrypted, it cannot be decrypted. Never use this type of field if you need to be able to get the password back out again.
A good example is the field `sys_user.password`. When you log into ServiceNow, it takes the password you entered on the login page and then encrypts it using the same encryption mechanism it used when you set your password, and then checks to see if the encrypted password you entered matches what is stored in your user's `sys_user.password` field.

```
Setting your password
New password: my_password
Encrypted: ePwaaS8VRwM=
Setting sys_user.password to ePwaaS8VRwM=

Logging in
Password given: my_password1234
Encrypted: bXlfcGFzc3dvcmQxMjM0
"ePwaaS8VRwM=" from the user is not the same as "bXlfcGFzc3dvcmQxMjM0" from the login page
Incorrect password
```

**Password (2 Way Encrypted)**
These are 2-way encrypted fields, where the data can be decrypted and retrieved to use in things like your custom code.
These sorts of fields are used all over the place, including:
* Discovery and Orchestration credentials.
* REST and SOAP authentication credentials.
* LDAP server passwords.

## Encrypting and Decrypting data
The magic behind the 2-way password fields is the `GlideEncrypter` class. You can use this to do things like encrypt and decrypt your own data, or decrypt data that's come from a Password 2 field.

Here's a quick demonstration of the class in action.

```js
var encr = new GlideEncrypter(); 
var clearString = 'my_password'; 
var encrString = encr.encrypt(clearString); // Something like 'ePwaaS8VRwM='
var decrString = encr.decrypt(encrString);  // 'my_password'
gs.print("Decrypted string = " + decrString);
```

Quickly decrypt something.

```js
new GlideEncrypter().decrypt(encryptedString)
```

You can use this same class to decrypt something from a 2-way password field. In this example, the password from an LDAP Server record.

```js
var encr = new GlideEncrypter();
var grLDAP = new GlideRecord("ldap_server_config");
grLDAP.get(); // Get one of them, I don't mind which
var clearString = encr.decrypt(grLDAP.password);
gs.print("Password for server "+grLDAP.getDisplayValue()+": "+clearString);
```

It's worth noting that this is still considered safe and secure, because the user must still have admin abilities to run such a script. This means that only admins or the ServiceNow system itself will be able to decrypt this information. Non-admins should not be able to run their own arbitrary scripts, and therefore should not be able to whimsically decrypt passwords.

As for 1-way encrypted data, I haven't yet found the mechanism that ServiceNow uses to encrypt data 1-way. However, you'd very rarely need to do this outside of storing the user's password.

More information about the `GlideEncrypter` class can be found here: https://developer.servicenow.com/dev.do#!/reference/api/orlando/server_legacy/GlideEncrypterAPI

ServiceNow HI has provided more information about encryption here in this ServiceNow Community discussion: https://community.servicenow.com/community?id=community_question&sys_id=b86c4064db79df402b6dfb651f96198e

> Password (1 Way Encrypted) Text field that stores passwords with one-way encryption. One-way encryption stores the password as a secure hash value that cannot be decrypted.
> Password (1 Way Encrypted) is using either of the three types.
> 
> Password (2 Way Encrypted) Text field that stores passwords with two-way encryption. Two-way encryption stores the password as a secure encrypted value that can be decrypted programmatically within the instance. You can use Password 2 encryption with form variables. To encrypt text fields on forms, use Encryption Contexts. The length for password2 field values must be at least 255 characters.
> Password (2 Way Encrypted) is using 256 AES.
> 
> Encryption key :
> Key used to encrypt the data. Leave this field blank to randomly generate a key. Based on the desired type of encryption, enter the exact number of characters:
> 
> * 24 characters for 3-key Triple DES
> * 16 characters for AES 128-bit
> * 32 characters for AES 256-bit (requires system configuration)

## ServiceNow encryption context
For those with interested in encryption mechanics, you'll know that you need an unchanging encryption key to reliably encrypt and decrypt data. For it to be unchanging, it needs to be stored somewhere. In my adventures I've discovered that this information is stored in the `sys_encryption_context` table, which is only accessible to users with the `maint` role.

Because the encryption context is locked away from non-ServiceNow users, it's unlikely that you'll be able to use this information, but it's cool to know nonetheless. 

I once encountered an issue on a freshly-cloned instance of ServiceNow where all passwords that were being decrypted came back as stars "\*\*\*\*\*\*". HI Support discovered that something had gone wrong with the recent clone, and that the encryption context hadn't been included in the clone, which meant that any encrypted data could not be encrypted. The issue was resolved by re-cloning over the instance again.