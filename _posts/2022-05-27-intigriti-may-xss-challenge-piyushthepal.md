---
layout: single
title: Intigriti's May XSS challenge By PiyushThePal
date: 2022-05-27
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - PiyushThePal
---

How far can you take prototype pollution? This challenge is a great showcase which uses an unpatched jQuery plugin to exploit and bypass two types of filters.

![share](/assets/images/intigriti/2022/05/share.jpg)

## Overview

The challenge functions on a single HTML page which changes content depending on a specific query parameter named “page”. The logic seems to be pretty straightforward. You have an object named “pages” which has four numeric attributes, each of them having plain html content as values. The weird part in all of this is the numeric naming convention used for the object attributes.

![vulnerable code](/assets/images/intigriti/2022/05/vulnerable-code.png)

It’s good to point out that due to heavy traffic on one of the CDNs, one of the libraries used for this challenge has been moved locally and pasted right inside the main page. I’m talking about the [jQuery.query](https://github.com/bmitchelmore/jquery.plugins/blob/main/jquery.query.js) plugin by Blair Mitchelmore.

Going back to the challenge, the logic seems to be pretty straightforward. You have an object named “pages” which has four numeric attributes, each of them having plain html content as values. The weird part in all of this is the numeric naming convention used for the object attributes.

![challenge page](/assets/images/intigriti/2022/05/challenge-page.png)

Upon loading the page, the jQuery.query plugin grabs the value of the query string parameter and checks whether the value matches some of the attributes in the `pages` object. If there’s a match, then the HTML contents will be displayed, otherwise it will load page `1` by default.

## Tampering

This all smells to me like prototype pollution. My first though was to try and use some of the object prototypes’ attributes. For example, using `__proto__` instead of a numeric value to to the query string.

[prototype](/assets/images/intigriti/2022/05/prototype.png)

My suspicions were correct! Now I have to find a way to pollute a value and get it displayed onto the page.

After a bit of googling I found [another repository](https://github.com/alrusdi/jquery-plugin-query-object) of the same jQuery plugin, but this one has documentation included. Here’s how it goes:

> Query String Modification and Creation for jQuery: This extension creates a singleton query string object for quick and readable query string modification and creation. This plugin provides a simple way of taking a page’s query string and creating a modified version of this with little code.

One feature that caught my attention was the ability to parse arrays from the query string.

```
?testy[]=true&testy[]=false&testy[]
```

The above query would result into an array with three boolean values.

Ok, so what’s the big deal here?

The deal here is that normally this kinds of transformation may be handled insecurely. Merging, cloning and parsing data into JavaScript objects is a risky business.

Moreover, I was able to immediately find the plugin listed in the “go-to” place for prototype pollution. I’m talking about the [Client-side prototype pollution](https://github.com/BlackFan/client-side-prototype-pollution) repository by BlackFan. There, the plugin has a [detailed description](https://github.com/BlackFan/client-side-prototype-pollution/blob/master/pp/jquery-query-object.md) and a PoC of the vulnerability. The key takeaway from this is that the vector `__proto__[test]=test` can be used to pollute objects.

## Exploitation

The weirdest part of this is that this is not an old version of the plugin. In fact, this is the current version and it has an active vulnerability..

Anyhow, here is my prototype pollution PoC on the challenge page.

![pollution](/assets/images/intigriti/2022/05/pollution.png)

Ok now all I have to do is to pollute the value with an XSS vector and all will be good, right? Well, no. There is another issue that is in the way.

![filter](/assets/images/intigriti/2022/05/filter.png)

However, the HTML value of each object attribute is being filtered by the [XSS node module](https://www.npmjs.com/package/xss). This module is pretty standard and doesn’t seem to have any documented bypasses.

I took it upon myself and went though the sanitizers’ codebase. After a while I found a particular case of a potential script gadget.

![fitlerxss](/assets/images/intigriti/2022/05/filterxss.png)

If the options object were to lack some attributes, then they would be assigned from the DEFAULT object instead.

![debug](/assets/images/intigriti/2022/05/debug.png)

After a quick debug, I indeed found out that the options object does not have any attributes initialized. This means that if I were to pollute it, then I would have control over the sanitization logic.

## Solution

Lets read more about the `whiteList` attribute. I found a documentation on [this GitHub link](https://github.com/leizongmin/js-xss).

> By specifying a `whiteList`, e.g. `{ 'tagName': [ 'attr-1', 'attr-2' ] }`. Tags and attributes not in the whitelist would be filter out.

I think that this would be just enough for me to inject XSS into the page. The only thing I would need to do is to pollute the whitelist like this:

```
__proto__[whiteList][svg][]=onload
```

I will then pollute once again, this time using my XSS vector:

```
__proto__[0]=<svg+onload%3Dalert(document.domain)>
```

Last but not least, I will set the `pages` value to `0`, so that my vector would get injected:

```
pages=0
```

Here is the complete string that I’ve constructed:

```
?__proto__[whiteList][svg][]=onload&__proto__[0]=<svg+onload%3Dalert(document.domain)>&page=0
```

Now let’s see if it works..

![alert](/assets/images/intigriti/2022/05/alert.png)

It works. Thanks for reading!