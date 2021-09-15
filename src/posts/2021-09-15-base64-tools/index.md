---
title: "Dave's Base64 Tools"
description: I use base64 a lot in my line of work, so I made my own tools to help me work faster!
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2021-09-15
tags:
- My creation
- Coding
eleventyExcludeFromCollections: false
---

**Link: https://towerofpower256.github.io/web-base64tools/**

* Encode string to base64
* Decode base64 back into string
* Encode a file to base64
* View a base64 picture

I use base64 a lot in my line of work, usually when I'm integrating 2 systems together and I'm transferring files or attachments.

Normally I'd use some online tools pretty frequently to encode stuff into base64 and decode them back out. Tools like:

* https://www.base64decode.org/
* https://www.base64decode.net/
* https://base64.guru/converter/encode

However, these tools send what I'm encoding or decoding **back to a server for processing**. Now that I do programming for a living and I sometimes work with confidential data, I'm not comfortable sending data like that back to someone else's server. Can they be trusted not to take a copy of the data? I don't know, and that's the conundrum.

So I wanted to make my own. All of the functions are written in Javascript and work on the user's browser. The data being encoded and decoded don't go anywhere else, and don't get sent to another server for processing. **It all happens on your computer.**

There was some fancy-dancing required for different character sets. Turns out that the native `atob()` and `btoa()` functions don't like characters that aren't a byte (8 bits) in size. Mozilla had a good workaround for this. https://developer.mozilla.org/en-US/docs/Glossary/Base64#the_unicode_problem

## What is base64 and when would I use it?
Converting something into base64 is to represent it as a simple string using only ASCII characters and no special characters like `\n` new-lines or `\t` tabs.

Example:
```
The quick brown fox jumps over the lazy dog.
```
encoded into base64 is
```
VGhlIHF1aWNrIGJyb3duIGZveCBqdW1wcyBvdmVyIHRoZSBsYXp5IGRvZy4=
```

Or something more complex, with some UTF-8:
```
✓ à la mode
```
encoded into base64 is
```
4pyTIMOgIGxhIG1vZGU=
```

This becomes useful when you have to transport something complex as simple text. In my line of work, this is usually when 
* integrating 2 systems together over REST or SOAP
* storing non-text data into a string field in a database
* storing non-text data into a JSON or XML payload
* embedding an image into an email or web page without a separate image file

Common examples of things to base64 encode:
* Files or attachments
* SSL certificates

There are more uses for base64, but these are the ones that come up most often for me.