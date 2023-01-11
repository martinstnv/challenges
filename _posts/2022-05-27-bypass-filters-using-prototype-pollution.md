---
layout: single
title: Bypass Filters using Prototype Pollution
date: 2022-05-27
classes: wide
tags:
  - Intigriti
  - Challenge
  - PiyushThePal
---

Use prototype pollution to bypass the "js-xss" sanitizer.

![share](/assets/images/intigriti/2022/05/share.jpg)

## Overview

Let's start off with the code. Here is what's important.

![vulnerable code](/assets/images/intigriti/2022/05/vulnerable-code.png)

There is an object named “pages” which has four numeric attributes, each of them having plain html content as values. The weird part in all of this is the numeric naming convention used for the object attributes.

Apart from this, the challenge has two dependencies - [jQuery.query](https://github.com/bmitchelmore/jquery.plugins/blob/main/jquery.query.js) and [js-xss](https://www.npmjs.com/package/xss). 

![challenge page](/assets/images/intigriti/2022/05/challenge-page.png)

The logic seems to be pretty straightforward. Upon loading the page, the jQuery.query plugin grabs the value of the query string parameter and checks whether the value matches some of the attributes in the `pages` object. If there’s a match, then the HTML contents will be displayed, otherwise it will load page `1` by default.

## Tampering

This all smells to me like prototype pollution. My first though was to use the prototype attribute instead of a numeric value.

![prototype](/assets/images/intigriti/2022/05/prototype.png)

Now I have to find a way to pollute a value and get it displayed onto the page.

After a bit of googling I found [another repository](https://github.com/alrusdi/jquery-plugin-query-object) for the same jQuery plugin, but this one has documentation included. Here’s how it goes:

> Query String Modification and Creation for jQuery: This extension creates a singleton query string object for quick and readable query string modification and creation. This plugin provides a simple way of taking a page’s query string and creating a modified version of this with little code.

One feature that caught my attention was the ability to parse arrays from the query string.

```
testy[]=true&testy[]=false&testy[]
```

The above query would result into an array with three boolean values. This gives me the ability to pollute attributes.

I was able to immediately find the plugin listed in the “go-to” place for [prototype pollution](https://github.com/BlackFan/client-side-prototype-pollution/blob/master/pp/jquery-query-object.md) which has a detailed description of the vulnerable component and aPoC:

```
__proto__[test]=test
```

The weirdest part of this is that this is the current version of the plugin.

## Exploitation

I craft a basic XSS vector and set pollute a the `0` page number:

```
?__proto__[0]=<svg+onload%3Dalert(document.domain)>&page=0
```

Here is the result:

![pollution](/assets/images/intigriti/2022/05/pollution.png)

However, here is another issue that is in the way.

![filter](/assets/images/intigriti/2022/05/filter.png)

The value of each attribute is filtered by the `js-xss` sanitizer.

I went though the sanitizer's codebase. After a while I found a particular case of a potential script gadget.

![fitlerxss](/assets/images/intigriti/2022/05/filterxss.png)

After a quick debug, I indeed found out that the options object does not have any attributes initialized. This means that if I were to pollute it, then I would have control over the sanitization logic.

![debug](/assets/images/intigriti/2022/05/debug.png)

## Solution

Lets read more about the `whiteList` attribute. I found a documentation on [this GitHub link](https://github.com/leizongmin/js-xss).

> By specifying a `whiteList`, e.g. `{ 'tagName': [ 'attr-1', 'attr-2' ] }`. Tags and attributes not in the whitelist would be filter out.

If possible, I could pollute the whitelist like this:

```
__proto__[whiteList][svg][]=onload
```

Here is the complete string that I’ve constructed:

```
?__proto__[whiteList][svg][]=onload&__proto__[0]=<svg+onload%3Dalert(document.domain)>&page=0
```

Now lets test this out.

![alert](/assets/images/intigriti/2022/05/alert.png)

It works.

Thanks for reading!
