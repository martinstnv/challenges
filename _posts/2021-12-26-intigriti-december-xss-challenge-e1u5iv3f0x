---
layout: single
title: Intigriti’s December XSS challenge By E1u5iv3F0x
date: 2021-12-26
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - E1u5iv3F0x
---

Weak referrer policy abd best-fit mapping transformations lead to 

![share](/assets/images/intigriti/2021/12/share.jpg)

## TL;DR

The solution that I came up with consists of two flaws: a weak referrer policy and a best-fit mapping transformation. If you are not familiar with best-fit mapping, here is a brief description from [Netsparker](https://www.invicti.com/web-vulnerability-scanner/vulnerabilities/unicode-transformation-best-fit-mapping/).

> A best-fit mapping occurs when one Unicode character is substituted by a similar looking character. This might happen, for example, if a Unicode string is converted to an ASCII string. Due to the vast differences in the amount of available characters, many encoding functions try to map complex Unicode characters to a similarly looking character in ASCII. Á becomes A, ò becomes o, and so on.

## Overview

The challenge seems simple at first glance. You click on a cracker until it breaks an reveals more elements.

![merry xmas](/assets/images/intigriti/2021/12/merry-xmas.png)

This is later skipped by adding a query parameter `open=true` (the value could be anything).

I was able to quickly identify one source of user input. It is the `payload` query parameter, which gets set after submitting a value inside the cracker’s hidden elements. The input is then reflected onto the page inside a heading.

Here an example result with the query parameters `?open=y&payload=example`:

![example payload](/assets/images/intigriti/2021/12/example-payload.png)

The next logical step is to try and inject HTML. However, it looks like there is a strong filter on the server-side. Everything between angle brackets is removed, including the brackets.

![filter](/assets/images/intigriti/2021/12/filter.png)

My next attempt is to use unicode characters. Sometimes the server side transforms unicode characters into regular ASCII. This is also known as the best-fit mapping, where unknown characters are matched with closest looking characters. I even found a great article on this subject. I chose to use a fullwidth [less-than](https://util.unicode.org/UnicodeJsps/character.jsp?a=FF1C) and [greater-than](https://util.unicode.org/UnicodeJsps/character.jsp?a=FF1E) signs.

Unfortunately, the result is not transformed into ASCII.

![best-fit mapping attempt](/assets/images/intigriti/2021/12/best-fit-mapping-attempt.png)

Here is where I am at a dead-end once again. Nothing came to mind that could work.

Fortunately, Intigriti’s tip came out and gave me a new idea!

![intigriti's tip](/assets/images/intigriti/2021/12/intigriti-tip.png)

I didn’t even notice that when adding a payload parameter that an HTML comment saying *“Referer: “* appears.

So, I made a quick exploit, in order to add value to the reflected referrer.

```html
<!DOCTYPE html>

<head>
    <script>
        if (location.href.indexOf("payload") < 0)
            location = location.href + '?payload=example';
    </script>
</head>

<body>
    <iframe src="https://challenge-1221.intigriti.io/challenge/index.php?payload=x&open=y"
        height="1000px" width="1000px"></iframe>
</body>

</html>
```

I used VS Code’s Live server extension to serve my PoC on port 5500. However, upon visiting `127.0.0.1:5500/index.html?payload=example` the result is missing the referrer’s path and is only showing the domain.

![example referrer payload](/assets/images/intigriti/2021/12/referrer-reflection.png)

I looked up information about iframe referrer policies and found [this page](https://www.w3schools.com/tags/att_iframe_referrerpolicy.asp). In the Attribute Values table there is an `unsafe-url` option which might just do the trick.

![referrerpolicy values](/assets/images/intigriti/2021/12/referrerpolicy-values.png)

Here is the result of adding `referrerpolicy=”unsafe-url”` to the frame.

![example referrer payload](/assets/images/intigriti/2021/12/example-referrer-payload.png)

Now, after injecting text, it’s time to try HTML as well. The payload is now changed to `127.0.0.1:5500/index.html?payload=--></h4><svg+onload=alert(document.domain)>`.

![encoded referrer payload](/assets/images/intigriti/2021/12/encoded-referrer-payload.png)

The brackets get encoded into HTML entities. This is troublesome since there is no way of injecting any kind of code with encoded brackets.

Here is where I go back to the best-fit mapping method. I modify my exploit to swap the angle bracket characters with the fullwidth signs from before.

```html
<!DOCTYPE html>

<head>
    <script>
        if (location.href.indexOf("payload") < 0)
            location = location.href + '?payload=--></h4><svg+onload=alert(document.domain)>'
                .replaceAll("<", "\uFF1C")
                .replaceAll(">", "\uFF1E");
    </script>
</head>

<body>
    <iframe
        src="https://challenge-1221.intigriti.io/challenge/index.php?payload=x&open=y" 
        referrerpolicy="unsafe-url"
        height="1000px" width="1000px"
    ></iframe>
</body>

</html>
```

The result is:

![injection](/assets/images/intigriti/2021/12/injection.png)

Which immediately triggers the alert upon visiting `http://127.0.0.1:5500/`.

![alert](/assets/images/intigriti/2021/12/alert.png)

Thanks for reading, I hope this was helpful!
