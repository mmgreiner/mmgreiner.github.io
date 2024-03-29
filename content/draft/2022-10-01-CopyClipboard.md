---
categories:
- programming
date: "2022-10-01T00:00:00Z"
draft: true
tags:
- JavaScript
title: JavaScript copy to clipboard
---

Links:

Good explanation: [How-to-copy-to-clipboard](https://stackabuse.com/how-to-copy-to-clipboard-in-javascript-with-the-clipboard-api/)

Sample code:

~~~javascript
function copyRichTextToClipboard(content) {
    const blob = new Blob([content], { type: "text/html" });
    const richTextInput = new ClipboardItem({ "text/html": blob });
    //navigator.clipboard.write([richTextInput]).then(() => { alert("Copied rich:" + content)});
    navigator.clipboard.write([richTextInput]).then(
        () => { }, 
        () => { console.log("failed to copy " + content); }
    );
}
~~~

Sample usage:

~~~html
<p id="anchors">
    <a href="https://www.sbb.ch/de">sbb 1</a>
    <a href="https://www.sbb.ch/de">sbb 2</a>
    <a href="https://www.sbb.ch/de">sbb 3</a>
    <a href="https://www.sbb.ch/de">sbb 4</a>
</p>
<button onclick="copyRichTextToClipboard(document.getElementById('anchors').innerHTML)">Copy several links</button>
~~~
