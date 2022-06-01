---
layout: layouts/post.njk
eleventyExcludeFromCollections: true
---
# Title 1
asd

## Title 2
asd

### Title 3
asd

#### Title 4
asd

I am normal
**I am bold**
*I am italic*
***I am italic and bold***

[I am a link](http://google.com)

http://google.com
www.google.com
test@email.com

![Test image](/assets/img/favicon.png)

List
* Item 1
* Item 2
    * Sub item 1
    * Sub item 2
* Item 3

Numbered list
1. Item 1
1. Item 2
    * Sub item 1
    * Sub item 2
1. Item 3

Quotes
> I am
> a quote
    >> And I am nested
> but I'm not nested
> look at me go!
> lolwut

Horizontal line
***

Inline code `someCode("lolwut")`

Code block
```js
function myAwesomeFunction() {
    var lolwut = "doobeedoo";

    return lolwut.length;
}
```


Table
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |

Details section, collapsible?
<details>
  <summary>Click to expand, test text!</summary>
  
  ### Heading
  1. A numbered
  2. list
     * With some
     * Sub bullets

```js
var lolwut = "asd";
```
</details>