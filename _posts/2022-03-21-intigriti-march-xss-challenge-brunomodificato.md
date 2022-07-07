---
layout: single
title: Intigriti’s March XSS Challenge By @BrunoModificato
date: 2022-03-21
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - aszx87410
---

This month’s challenge was a bit tricky, but still fun nonetheless. The main goal is to bypass CSP protection **in a way that is not mentioned elsewhere**. Apart from that the challenge offers two more barriers - a CSRF token and an XSS filter. If you’re interested how this challenge was solved, I suggest you to continue reading!

![share](/assets/images/intigriti/2022/03/share.jpg)

## Overview

The challenge starts off with a simple form that submits plain text and a hash algorithm to the server. The path points to a page named `LoveSender.php` which is obviously PHP.

![LoveSender](/assets/images/intigriti/2022/03/lovesender.png)

Upon inspecting the source code, I found that there is another field which was hidden.

![hidden token field](/assets/images/intigriti/2022/03/hidden-token-field.png)

The name of the field is “token” which hints that it might be used as a CSRF protection token. The value changes upon reload as well.

Apart from the hidden field there is not much going on in this page, so I tried to submit the form with somewhat valid data.

![testing LoveSender](/assets/images/intigriti/2022/03/testing-lovesender.png)

The result is displayed on another page named `LoveReceiver.php`.

![LoveReceiver](/assets/images/intigriti/2022/03/lovereceiver.png)

It reflects back the plain text and uses the hash argument to return a hashed value. The hash value is valid which is bad because it cannot be tampered with in order to benefit the XSS challenge.

Fortunately, the plain text value is reflected without any changes. However, as the page suggests, there is some kind of a filter stopping me from using parenthesis and backticks.

The filter is actually pretty easy to bypass since there are a lot of examples that can be used. The first one that came to mind was the classic example of assigning the alert function to the window error handler:

```
onerror=alert; throw document.domain;
```

This example is successful at bypassing the filter, but there is another one which came to mind after seeing Intigriti’s second hint.

![second intigriti hint](/assets/images/intigriti/2022/03/second-hint.png)

Simply googling “reserved html characters” gives me the answer.

![reserved html characters](/assets/images/intigriti/2022/03/reserved-html-characters.png)

The answer is .. HTML entities!

Using HTML entities that interpret parenthesis is the more elegant way of bypassing the filter. Here is an example:

```
<svg onload=alert&lpar;document.domain&rpar;>
```

Now when that’s out of the way let’s see if the alert get’s executed.

If only it was this easy…

![blocked by csp](/assets/images/intigriti/2022/03/blocked-by-csp.png)

There is a content security policy put in place. If I review the page’s response I can see the full policy.

```
default-src 'none';
style-src 'nonce-16849b0d8743223d60cc7752f6f7abbbd1330e91';
script-src 'nonce-16849b0d8743223d60cc7752f6f7abbbd1330e91';
img-src 'self'
```

If you still don’t understand the scope of it the you can use Google’s CSP Evaluator to help you out!

![csp evaluator](/assets/images/intigriti/2022/03/csp-evaluator.png)

The Evaluator says that a missing “base-uri” policy might give an attacker control over all scripts that are using relative paths. Well, guess what? There aren’t any… This policy well done and cannot be bypassed. Trust me I tried.. for about two days.

Luckily, I had another option which came late to mine (two days late) and that was to follow Intigriti’s tips..

The first hint is about PHP having to do something with size. I have no idea what this might be..

![first intigriti hint](/assets/images/intigriti/2022/03/first-hint.png)

The third hint is again somewhat connected to the CSP problem.

![third hint](/assets/images/intigriti/2022/03/third-hint.png)

Go around it? I remember on an old challenge where the solution was to make the CSP stronger in order to bypass it. This is not the case here! Addition to the CSP only make it worse!

Another thing that I forgot to mention is that whenever I pass a bad value as a hash algorithm argument, the resulting page returns these errors:

![error messages](/assets/images/intigriti/2022/03/error-messages.png)

This only shows up on top of the page and the rest of it, apart from the hash result, remain as usual.

The error doesn’t seem to be that helpful as well..

So, I tried to follow Intigriti’s first tip, which is about PHP and size.

I passed inputs of great length inside the plain text field but the page managed it without any issues. However, doing the same to the hash argument is a different story!

I passed an input of about a thousand "A" characters to the hash field and viola! I’ve got a new error message at the bottom!

![new error message](/assets/images/intigriti/2022/03/new-error-message.png)

The message states the following:

> **Warning:** Cannot modify header information - headers already sent by (output started at /var/www/html/challenge/LoveReceiver.php:25) in **/var/www/html/challenge/LoveReceiver.php** on line **44**

If you’re still confused what this means, let me explain..

CSP headers are not sent anymore!!!

If you’re wondering how is that possible, the answer is **overflowing the PHP buffer**. The server is sending the response before the CSP headers are set. In the PHP manual there is a [brief description about output buffering](https://www.php.net/manual/en/outcontrol.configuration.php#ini.output-buffering):

> You can enable output buffering for all files by setting this directive to ‘On’. If you wish to limit the size of the buffer to a certain size — you can use a maximum number of bytes instead of ‘On’, as a value for this directive (e.g., output_buffering=4096). This directive is always Off in PHP-CLI.

What this means is that the default size of the PHP buffer under most configurations is 4096 bytes (4KB). In our case, due to overflowing the output buffer, the code is sending the response back to the client before executing the part where CSP headers are set.

Now when that’s out of the way lets try to execute our payload with the new trick in place.

![fuzzing hash field](/assets/images/intigriti/2022/03/fuzzing-hash-field.png)

The result of this being:

![self xss alert](/assets/images/intigriti/2022/03/self-xss-alert.png)

However the challenge is not over yet!

If you pay close attention to the request that is made in order to trigger the payload, you will find that it is a POST request. There are no query string parameters to ease things up and the GET request does not work as well.

In order to execute this payload, a fake website should be put in place to trigger the POST request. A nice trick to use is this piece of code:

```
<form id='poc' target='_blank'  method='POST' action='//victim.com'>
  <input type='text' name='payload' value='malicious payload' >
  <input type='submit'>
</form>
<script>
  poc.submit();
</script>
```

After trying this technique I get the following result:

![missing csrf token](/assets/images/intigriti/2022/03/missing-csrf-token.png)

I forgot about the CSRF token!

One thing was strange however. Upon testing different payloads before I got to this point I had to resend the POST request many times using the same CSRF token!

If you are unaware, a CSRF should be usable only once, otherwise it loses it’s purpose! This means that I could simply grab one token from the sender page and use it as much as I like.

I also saw Intigriti’s last hint which was about CSRF:

![fourth hint](/assets/images/intigriti/2022/03/fourth-hint.png)

So, I was right after all! Moreover, not only that there’s no proper token validation, but the only condition that makes a token valid is the length.

In order to test this out, I unhide the token field manually from the sender form and set its value to a random number of characters.

![fuzzing csrf token length](/assets/images/intigriti/2022/03/fuzzing-csrf-token-length.png)

I know that the correct length is 64 characters, judging by the length of an original token taken from the page. When I enter a token with incorrect length I get an “Invalid token!” response.

![invalid token](/assets/images/intigriti/2022/03/invalid-token.png)

Ok now back to the fake website.. I now execute my payload and get the following result:

![again missing csrf token](/assets/images/intigriti/2022/03/missing-csrf-token.png)

WHAT? Oh there is a mention about sending at least one request from the gui. Ok I think I can automate that as well…

Soo, I made a quick script which pops up the sender page, closes is and then makes the post request. It doesn’t work on the first try for some reason, so I made it reload after each attempt, no matter if it’s successful or not. I also made a 5 second interval between the sender page and the POST request, just to make sure that the sender page gets loaded properly. In total it should take a minimum of ~11 seconds to trigger the alert. The whole process does not require any user interaction, apart from enabling pop-ups.

```html
<form id="my-form" target="_blank"  method="POST" action="https://challenge-0322.intigriti.io/challenge/LoveReceiver.php">
    <input type="text"    id='token'       name="token">
    <input type="text"    id='FirstText'   name="FirstText">
    <input type="text"    id='Hashing'     name="Hashing">
    <input type="submit">
</form>

<script>

    let popup = window.open("https://challenge-0322.intigriti.io/challenge/LoveSender.php", "_blank");
    
    const payload = "<svg onload='alert&lpar;document.domain&rpar;'>";
    const varcharOverflow = "A".repeat(1000);
    const mockToken = 'A'.repeat(64);

    document.getElementById("token").value = mockToken;
    document.getElementById("FirstText").value = payload;
    document.getElementById("Hashing").value = varcharOverflow;

    setTimeout(() => {

        popup.close();
        document.getElementById("my-form").submit();

        setTimeout(() => {
            location.reload();
        },1000)
    }, 5000);

</script>
```

Finally, let’s test this out!

![alert](/assets/images/intigriti/2022/03/alert.png)

It works on both Firefox and Chrome. The solution is kindly accepted by PinkDraconian.

Thanks for reading! :)