---
layout: single
title: Intigriti’s October XSS Challenge by 0xTib3rius
date: 2021-10-25
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - 0xTib3rius
---

Broken syntax and weird browser behavior lead to cross-site scripting. This is my solution to the Halloween edition of Intigriti’s XSS challenge!

![share](/assets/images/intigriti/2021/10/share.jpg)

## Overview

What caught my attention immediately was the script in the end of the source code. 

![vulnerable code](/assets/images/intigriti/2021/10/vulnerable-code.png)

It seems to generate another script, however the syntax looks broken and the end result looks something like this:

![broken script](/assets/images/intigriti/2021/10/broken-script.png)

The `null` part is because of a missing query string parameter, called `xss`, which is appended to the script text.

Another thing to add is that the page has uses CSP.

```
default-src 'none';
script-src 'unsafe-eval' 'strict-dynamic' 'nonce-...';
style-src 'nonce-...'
```

This means that direct injections to the page would not work unless they have a nonce.

Going back to the previous snippet, the new script is made using `document.createElement(“script”)` which is an exception to the `strict-dynamic` rule and therefore bypasses it.

Now all I need is to find a way to fix the syntax.

## Solution

Now, let’s take a deep dive on how the script gets generated.

It takes the last child element of the document body and checks if it’s ID is equal to “intigriti”. Then it takes its’ last child element and copies the last 4 characters of it’s inner HTML contents into the beginning of the newly generated script.

This seems pretty self-explanatory. I need to somehow manipulate the ID the last child element of the document and control it’s inner elements to fix the broken script. However there are many elements in between that make this task really challenging.

As you may know, the browser does not accept broken HTML and always “tries” to fix, it in order to render the page. Those fixes are often pretty strange and might seem illogical to some, however this is what I used to solve the challenge.

![example injection](/assets/images/intigriti/2021/10/example-injection.png)

After a few hours of playing around and trying break the page in a way that would be useful to me, I’ve finally found a way.

Here is the payload that I’ve used, which takes advantage of unclosed HTML tags `html=</h1></div><object+id=intigriti><object><object'1` together with the actual payload `xss=-alert(document.domain)` which results to:

![malicious injection](/assets/images/intigriti/2021/10/malicious-injection.png)

The injection is successful.

![alert](/assets/images/intigriti/2021/10/alert.png)

Thanks for reading!