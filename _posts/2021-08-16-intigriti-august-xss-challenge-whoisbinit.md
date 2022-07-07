---
layout: single
title: Intigriti’s August XSS Challenge by WHOISbinit
date: 2021-08-16
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - WHOISbinit
---

This challenge is a great introduction to prototype pollution.

![share](/assets/images/intigriti/2021/08/share.jpeg)

In memory of Binit Ghimire, who passed away on the 25th on June 2022.

## Overview

Initially, a welcome message appears on the page with what seems to be a randomly generated username. This value could also be controlled via the cookie `username`. Upon loading the browser window, a custom script checks if such a cookie exists, otherwise it sets the random value.

Next, the script uses a the `deparam` function from the "jQuery Deparam" plugin, and passes the value of the `recipe` query string parameter into it. There are also a few recipe examples on the main page.

The expression is later followed by the `ga` function, which creates a google analytics cookie. [Here is the official reference](https://developers.google.com/analytics/devguides/collection/analyticsjs/cookies-user-id) to the function.

And lastly it calls upon the `welcomeUser` function, passing the username cookie value as an argument.

![vulnerable code](/assets/images/intigriti/2021/08/vulnerable-code.png)

The function has a dangerous `innerHTML` assignment which could lead to an injection.

![dangerous function](/assets/images/intigriti/2021/08/dangerous-function.png)

Another odd part is the comment above the vulnerable function:

> As we are a professional company offering XSS recipes, we want to create a nice user experience where the user can have a cool name that is shown on the screen Our advisors say that it should be editable through the web interface but I think our users are smart enough to just edit it in the cookies. This way no XSS will ever be possible because you cannot change the cookie unless you do it yourself!

This would prove useful later on!

## Solution

After some googling for any particular vulnerabilities for the jquery-deparam script I found an awesome [repository](https://github.com/BlackFan/client-side-prototype-pollution) for client-side prototype pollution by [BlackFan](https://github.com/BlackFan). It turns out that the script is really old and vulnerable to prototype pollution (CVE-2021–20087).

Moreover, the google analytics script is also mentioned in the repository, that it could be used as a script gadget for the prototype pollution vulnerability.

So it turns out that I can pollute the prototype object and rewrite the `cookieName` attribute that affects the Google Analytics script.

After decoding one of the example queries from the main page, I find more query parameters:

![example payload](/assets/images/intigriti/2021/08/example-payload.png)

Next, I append the polluted `cookieName` attribute and set the value of the `username` cookie to `<img src onerror=alert(document.domain)>`.

```
&__proto__[cookieName]=username%3D<img+src+onerror%3Dalert(document.domain)>%3B
```

*Note*: I should encode the result back to base64 and replace it in the URL

Now here is the tricky part, this does work but it is not the correct solution. What happens is that the XSS is only triggered when I refresh the page three times. Why so? This is because we are unable to override the previous cookie that is set with a random value just before the GA script. Only after the third time I reload the page, does the original cookie disappear.

Is there any other way to prioritize the modified cookie over the original?

The answer is yes and it was thanks to the people in Intigriti’s discord channel that lead me on the right track to take a look at [RFC 2109](https://www.ietf.org/rfc/rfc2109.txt). The document states the following:

> Note that the NAME=VALUE pair for the cookie with the more specific Path attribute, /acme/ammo, comes before the one with the less specific Path attribute, /acme. Further note that the same cookie name appears more than once.

What this means is that if I give the modified cookie a more precise path than the original one, it will be prioritized and there would be no of any override reloads, nor user interaction.

```
&__proto__[cookiePath]=/challenge/cooking.html
```

Here is the complete payload to the challenge and it’s corresponding result:

```
dGl0bGU9VGhlJTIwU1ZHJmluZ3JlZGllbnRzJTVCJTVEPUFuJTIwU1ZHJTIwdGFnJmluZ3JlZGllbnRzJTVCJTVEPVNvbWUlMjBqYXZhc2NyaXB0JmluZ3JlZGllbnRzJTVCJTVEPVNvbWUlMjBvbmxvYWQlMjBhY3Rpb24mcGF5bG9hZD0lM0NzdmclMjBvbmxvYWQlMjUzRGFsZXJ0KDEpJTNFJnN0ZXBzJTVCJTVEPUZpbmQlMjB0YXJnZXQmc3RlcHMlNUIlNUQ9SW5qZWN0JnN0ZXBzJTVCJTVEPUFMRVJUISZfX3Byb3RvX19bY29va2llTmFtZV09dXNlcm5hbWUlM0Q8aW1nK3NyYytvbmVycm9yJTNEYWxlcnQoZG9jdW1lbnQuZG9tYWluKT4lM0ImX19wcm90b19fW2Nvb2tpZVBhdGhdPS9jaGFsbGVuZ2UvY29va2luZy5odG1s
```

Now, to test this out!

![alert](/assets/images/intigriti/2021/08/alert.png)

As expected, it works.

Thanks for reading!