---
title: "Multi-Line Strings in Arduino"
description: I've been playing around with doing Web servers on ESP8266 wifi microcontrollers and I've run into a tedious problem of how to write large blocks of text like HTML and CSS as strings within the C++ code. Here are some options that I've found and how well they work.
image: featured.jpg
imageThumbnail: featured-thumbnail.jpg
date: 2024-12-07
tags:
- Arduino
- Coding
eleventyExcludeFromCollections: false
---

## The problem
What's the best way to write a HTML inside of your C / C++ code?

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Hello world!</title>
    </head>
    <body>
        <p>
            Hello world!
        </p>
    </body>
</html>
```

Or this help text for an interactive serial interface?

```
usage: app_name [options] required_input required_input2
  options:
    -a, --argument     Does something
    -b required     Does something with "required"
    -c, --command required     Something else
    -d [optlistitem1 optlistitem 2 ... ]     Something with list
```

## Method 1 - Raw String Literals
This method involves using keywords at the start and end of the string. Anything between those keywords will be compiled as the string, even across multiple lines. 

Only works for compilers C++11 and later.

```cpp
const char * mystring = R"ILUVDISCO( 
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Hello world!</title>
    </head>
    <body>
        <p>
            Hello world!
        </p>
    </body>
</html>
)ILUVDISCO";

Serial.print(mystring);
```

The keyword can be whatever you like. 

```cpp
const char * mystring1 = R"THISWORKS(
    This works
)THISWORKS";

const char * mystring2 = R"===(
    So does this
)===";

const char * mystring3 = R"r0ck_th3-house(
    This works too
)r0ck_th3-house";
```

The good
* Don't need to add anything to your text, don't need to wrap in quotation or brackets or backslashes.
* You can copy-paste text straight into your code.
* Easier to read in code.

The bad
* Only supported on C++11 compilers or newer (but that's most of them).

## Method 2 - backslash newline 
You can keep writing a string wrapped in quotes across multiple lines by ending each line with a "\" backslash. This means "include this following new-line character as part of the string".

```cpp
const char * mystring = "<!DOCTYPE html> \
<html lang=\"en\"> \
    <head> \
        <title>Hello world!</title> \
    </head> \
    <body> \
        <p> \
            Hello world! \
        </p> \
    </body> \
</html>";

Serial.print(mystring);
```

The good
* Less tedious then multiple string litterals, each line just needs to end with a "\" backslash instead of brackets and a function call.
* Interpreted as 1 large string.

The bad
* Still a little tedious because you need to end each line with a "\" backslash. 
* Can't just copy-paste a large string into your code.

## Method 3 - multiple quoted strings
You can have multiple strings wrapped in quotation stacked next to each other in a single call and the compiler will interpret it as 1 single string.

```cpp
const char * mystring = 
"<!DOCTYPE html>"
"<html lang=\"en\">"
"<head>"
"<title>Hello world!</title>"
"</head>"
"<body>"
"<p>"
"Hello world!"
"</p>"
"</body>"
"</html>";

Serial.print(mystring);
```

The good
* Interpreted as 1 large string.

The bad
* Still tedious, each line needs to be wrapped in quotation.
* Can't just copy-paste a large string into your code.

## Method 4 - multiple output statements
The easy option is to simply call your output function multiple times with multiple single-line strings, 1 for each line.

```cpp
Serial.print("<!DOCTYPE html>");
Serial.print("<html lang=\"en\">");
Serial.print("<head>");
Serial.print("<title>Hello world!</title>");
Serial.print("</head>");
Serial.print("<body>");
Serial.print("<p>");
Serial.print("Hello world!");
Serial.print("</p>");
Serial.print("</body>");
Serial.print("</html>");
```

The good
* Works OK for small multiline strings e.g. 2 or 3 lines.

The bad
* Tedious for many lines, you need to wrap brackets and function calls around every line.
* May introduce additional processing overhead depending on the output function.
* Can't just copy-paste a large string into your code.

## Method 5 - Concatenated Strings
I don't recommend this, but you can chain Strings together to make 1 large string.

```cpp
String mystring = String("<!DOCTYPE html>");
mystring += "<html lang=\"en\">";
mystring += "<head>";
mystring += "<title>Hello world!</title>";
mystring += "</head>";
mystring += "<body>";
mystring += "<p>";
mystring += "Hello world!";
mystring += "</p>";
mystring += "</body>";
mystring += "</html>";

Serial.print(mystring);
```

I don't recommend you do this. There's ongoing concern about the Arduino ```String``` class using dynamic memory allocation, which has been known for causing memory fragmentation issues, resulting in crashes & resets. I've had it happen to me a few times, I try not to use them if I can avoid it.

* [Forward - Taming Arduino Strings Avoiding Fragementation and Out-of-Memory Issues](https://www.forward.com.au/pfod/ArduinoProgramming/ArduinoStrings/index.html#stringErrors)
* [Instructables - Taming Arduino Strings -- How to Avoid Memory Issues](https://www.instructables.com/Taming-Arduino-Strings-How-to-Avoid-Memory-Issues/)
* [Arduino Forum - Is String still bad?](https://forum.arduino.cc/t/is-string-still-bad/351631)
* [Arduino Forum - Understanding the evil String()](https://forum.arduino.cc/t/understanding-the-evil-string/427528)
* [xenForo Forum - String class warning on Arduino forum](https://forum.pjrc.com/index.php?threads/string-class-warning-on-arduino-forum.62859/)
* [Arduino Forum - The HATRED for String objects - "To String, or not to String"](https://forum.arduino.cc/t/the-hatred-for-string-objects-to-string-or-not-to-string/121801/5)

The good
* Interpreted as 1 large string.

The bad
* Using Arduino `Strings` can be dangerous.
* Tedious to write in your code, need to wrap each string in quotation and your String variable.
* Can't just copy and copy-paste text into your code.

## The better answer - put it in a file
All of these suggestions are how to include big blocks of text as a variable within your code.

However, any text larger than a handful of lines (e.g. 20 lines) should likely be:
* moved into their own file (e.g. `index.html`)
* called as the file from within your code

There are many ways to do this and deserve their own article on how to do this with an Arduino or ESP8266.