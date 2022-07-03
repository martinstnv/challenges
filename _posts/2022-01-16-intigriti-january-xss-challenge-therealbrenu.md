---
layout: single
title: Intigriti’s January XSS Challenge By TheRealBrenu
date: 2022-01-16
classes: wide
tags:
  - Intigriti
  - XSS
  - Challenge
  - TheRealBrenu
---

First challenge for 2022 is here by TheRealBrenu. This one is a good example of javascript source maps, which I was unfamiliar at first. However, more on that later.

![share](/assets/images/intigriti/2022/01/share.jpg)

## Overview
The challenge starts off as a simple page with nothing more than a simple text area and a button. You can submit some text and it will render it out on another page with the path `result` and a query string parameter `payload`.

After a quick source code inspection I find that there is only one script source which seems to be minified.

![example payload](/assets/images/intigriti/2022/01/example-payload.png)

The only thing that stood out to me was a comment in the beginning saying

> For license information please see main.02a05519.js.LICENSE.txt

I opened the suggested license file and found out that the app is using DOMPurify 2.3.4, which is currently the latest version and there was no point in trying to hack that. So, I went to check the sources next.

There, I immediately found out that the challenge is using React. Luckily, I am a former React developer and this was music to my ears.

![exposed source map](/assets/images/intigriti/2022/01/exposed-source-map.png)

I was a bit confused that the code was exposed even though there was a minified bundle. After Intigriti gave out the second hint I managed to dig in more on the topic what and why this happened.

![intigriti tip](/assets/images/intigriti/2022/01/intigriti-tip.png)

The hint itself wasn’t very helpful since I have already submitted my solution before then. However I did a bit of digging and found an [article by Ryan Seddon](https://developer.chrome.com/blog/sourcemaps/) which explained it perfectly.

> Basically it’s a way to map a combined/minified file back to an unbuilt state. When you build for production, along with minifying and combining your JavaScript files, you generate a source map which holds information about your original files. When you query a certain line and column number in your generated JavaScript you can do a lookup in the source map which returns the original location. Developer tools (currently WebKit nightly builds, Google Chrome, or Firefox 23+) can parse the source map automatically and make it appear as though you’re running unminified and uncombined files.

So, even though I wasn’t aware of source maps, just by looking at all the sources I managed to puzzle the pieces together.

In a file named `router.js` I found an `identifiers` object which had base64 encoded values. The object is then passed as `props` to two components located in the `pages` folder. You can find the full code here.

I managed to decode all values via the console.

![decoded identifiers](/assets/images/intigriti/2022/01/decoded-identifiers.png)

Here I see some important keywords mentioned, such as `payloadFromUrl`, `data-debug`, `sanitizeHTML`, `sanitize`, where the sanitize keywords were obviously referring to DOMPurify.

Upon looking at the components in the pages folder, I find that most of the code is not really readable. I get it that this is used for some kind of obfuscation. I was more interested in the component that loaded on the `result` path. The name of this component was obfuscated as well but I supposed it was `I0x1`, which was mapped to `Result`.

```jsx
function I0x1({ identifiers }) {
  const [I0x2, _] = useState(() => {
    const I0x3 = new URLSearchParams(
      window[window.atob(identifiers["I0x4"])][window.atob(identifiers["I0x5"])]
    )[window.atob(identifiers["I0x6"])](window.atob(identifiers["I0x7"]));

    if (I0x3) {
      const I0x8 = {};
      I0x8[window.atob(identifiers["I0x9"])] = I0x3;

      return I0x8;
    }

    const I0x8 = {};
    I0x8[window.atob(identifiers["I0x9"])] = window.atob(identifiers["I0xA"]);

    return I0x8;
  });

  function I0xB(I0xC) {
    for (const I0xD of I0xC[window.atob(identifiers["I0xE"])]) {
      if (
        window.atob(identifiers["I0x11"]) in
        I0xD[window.atob(identifiers["I0xF"])]
      ) {
        new Function(
          I0xD[window.atob(identifiers["I0x10"])](
            window.atob(identifiers["I0x11"])
          )
        )();
      }

      I0xB(I0xD);
    }
  }

  function I0x12(I0x13) {
    I0x13[window.atob(identifiers["I0x9"])] = DOMPurify[
      window.atob(identifiers["I0x15"])
    ](I0x13[window.atob(identifiers["I0x9"])]);

    let I0x14 = document[window.atob(identifiers["I0x16"])](
      window.atob(identifiers["I0x14"])
    );
    I0x14[window.atob(identifiers["I0x17"])] =
      I0x13[window.atob(identifiers["I0x9"])];
    document[window.atob(identifiers["I0x32"])][
      window.atob(identifiers["I0x18"])
    ](I0x14);

    I0x14 = document[window.atob(identifiers["I0x19"])](
      window.atob(identifiers["I0x14"])
    )[0];
    I0xB(I0x14[window.atob(identifiers["I0x1A"])]);

    document[window.atob(identifiers["I0x32"])][
      window.atob(identifiers["I0x1B"])
    ](I0x14);

    return I0x13;
  }

  return (
    <div className="App">
      <h1>Here is the result!</h1>
      <div id="viewer-container" dangerouslySetInnerHTML={I0x12(I0x2)}></div>
    </div>
  );
}
```

What I did from here on out was to manually swap all keywords to their corresponding places. I didn't bother to automate the process and did it by hand which took about 15 minutes.

Here is the de-obfuscated result:

```jsx
function Result() {

    const [payloadFromUrl, _] = useState(() => {
    const queryResult = new URLSearchParams(window.location.search).get("payload");

    if (queryResult) {
      const result = {};
      result.__html = queryResult;

      return result;
    }

    const result = {};
    result.__html = "<h1 style='color: #00bfa5'>Nothing here!</h1>";

    return result;
  });

  function handleAttributes(element) {
    for (const child of element.children) {
      if ("data-debug" in child.attributes ) {
        new Function(child.getAttribute("data-debug"))();
      }

      handleAttributes(child);
    }
  }

  function sanitizeHTML(htmlObj) {
    htmlObj.__html = DOMPurify.sanitize(htmlObj.__html);

    let template = document.createElement("template");
    template.innerHTML = htmlObj.__html;
    document.body.appendChild(template);

    template = document.getElementsByTagName("template")[0];

    handleAttributes(template.content);

    document.body.removeChild(template);

    return htmlObj;
  }

  return (
    <div className="App">
      <h1>Here is the result!</h1>
      <div id="viewer-container" dangerouslySetInnerHTML={sanitizeHTML(payloadFromUrl)}></div>
    </div>
  );
}
```

The `handleAttributes` seems a bit dangerous. The method dives in recursively into all child attributes and if they correspond to the “data-debug" keyword, the values would be passed as arguments to a Function constructor.

Most people aware of XSS might know that the `Function` constructor is actually a dangerous function and will evaluate string arguments.

```js
function handleAttributes(element) {
  for (const child of element.children) {
    if ("data-debug" in child.attributes ) {
      new Function(child.getAttribute("data-debug"))();
    }

    handleAttributes(child);
  }
}
```
So now I just need to craft a quick payload like `/result?payload=<i+data-debug=alert(document.domain)>` and hopefully all will go as planned!

![alert](/assets/images/intigriti/2022/01/alert.png)

Here we go!

Thanks for reading!