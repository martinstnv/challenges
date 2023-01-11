---
layout: single
title: Using Javascript Labels to Bypass Length Restrictions 
date: 2022-02-07
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - aszx87410
---

Javascript labels and SVG elements to execute arbitrary code in Firefox.

![share](/assets/images/intigriti/2022/02/share.jpg)

## Overview

The challenge is represented by an HTML form which collects two things - the name of the user and an answer to “Have you even played this game?”. Upon submitting the form the page is reloaded with the values of the fields being passed as query string parameters. The page then checks if the parameters are in place and pops up a modal with the name being dangerously set as an inner HTML element.

![challenge](/assets/images/intigriti/2022/02/challenge.png)

The key point here is that the payload should be no longer than 24 characters and at the same time trigger `alert(document.domain)` which by itself is 22 characters long.

## Solution

Judging by the fact that one of the shortest XSS payloads is `<svg/onload=eval()>` which is 19 characters long, then we are left with only 5 characters to use.

![filtered global variables](/assets/images/intigriti/2022/02/filtered-global-variables.png)

By using a small script I was able to enumerate how many global attributes had length less than 6 characters. My first bet was the name attribute but unfortunately it already has an assigned value.

What caught my eye next were the custom attributes `uri` and `qs` which are initialized by page. The `qs` attribute holds the value of the query string parameter which I am already using to send my HTML injection and trying to use it would defy logic. However, the `uri` parameter holds the value of `location.href` and this is something that I could use to slip in the malicious payload.

From previous experience with javascript I my immediate thought about this is that *“no way this could be considered valid javascript syntax because it starts with https:”*. To my luck I was wrong and googling lead me to the discovery of [JavaScript labels](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label). Here is a nice quote from JavaPoint:

> JavaScript label is a **statement used to prefix a label as an identifier**. You can specify the label by any name other than the reserved words. It is simply used with a colon (:) in code. A label can be used with a break or continue statement to control the flow of the code more precisely.

So, in my case https: is considered as a label and the backslashes turn the URL into a commented out code. The only thing that I have to do now is break out of the comment by adding a new line and simply add the payload that I want. In order to embed a new line into the URL it needs to be encoded as `%0A`.

![alert](/assets/images/intigriti/2022/02/alert.png)

It seems to work perfectly! Should I submit now? Of course! .. not …

Why is this not okay? The XSS is working as expected but just before submitting my solution I saw that Intigriti rejected a few solution due to the face that they do not work on Firefox!

I immediately tried my solution of Firefox and to my surprise it DIDN’T work… Why was this happening?

After a bit of googling I found out that Firefox has some complications with the load event on SVG elements.

![browser compatibility](/assets/images/intigriti/2022/02/browser-compatibility.png)

It appears that Firefox uses an alternate name for the `load` event which is `SVGLoad`. Now this is a problem since it will again exceed the character limit of my injection.

So are there any other elements that could be used instead of SVG? The answer is YES and here is a table:

![elements with load events](/assets/images/intigriti/2022/02/elements-with-load-events.png)

I didn’t think much of it and immediately grabbed the style element. Now the payload is a bit longer but still manages to not exceed the character limit. 

Here is how it looks like:

```
<style/onload=eval(uri)>&first=yes%0Aalert(document.domain)
```

Luckily this works like a charm and Firefox is able to trigger the alert.

![alert on firefox](/assets/images/intigriti/2022/02/alert-firefox.png)

Thanks for reading!

