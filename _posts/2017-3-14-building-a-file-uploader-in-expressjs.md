---
layout: post
title: Building a Progressive File Uploader in EmberJS
---

### Introduction
Uploading files is tricky (mostly conceptually), because it requires both front-end and back-end systems to be in sync. Building a system for uploading
files and *tracking the progress* is even harder. And finally, building a *resumable* uploader is quite a challenge. In this tutorial,
I will be building a progressive file uploader using **NodeJS** and **ExpressJS**. However, the concepts provided
in this tutorial should easily translate to other languages and / or frameworks.

### Scaffolding out an ExpressJS server
For starters, we need to have a webserver capable of deploying client side Javascript and running back-end Javascript. [ExpressJS](https://expressjs.com/)
comes prepackaged with a quick generator for small applications - so lets use that.

In order to use the ExpressJS generator, we will need both NodeJS and NPM installed locally. I am running NodeJS 7.7.0 and NPM 4.1.2 
for this tutorial. But you could get away with using older versions of both if you prefer, as long as they are above 6.4.0 (basic ES6 support).

Once you have both NodeJS and NPM installed, open up your terminal and write the following commands:

```
npm install -g expressjs
express --view=hbs --css=less uploader
```

This installs ExpressJS globally on your machine, and than generates an ExpressJS application scaffold with handlebars templating
and the LESS CSS preprocessor.

Navigate into that folder and install dependencies:

```
cd uploader
npm install
```

Now navigate to http://localhost:3000 - you should see a "Welcome to ExpressJS" message. This is where you will test your code.

If we browse through the files created by the ExpressJS generator, we will notice a few folders in the heirarchy. Right now we are 
interested in the index file under "views". This is the default view which will be loaded when a user visits www.your-url.com.

Inside of this file, delete the markup and replace it with:

```html
<h2>File Uploader</h2>
<div class="progress">
    <div class="progress-bar" role="progressbar"></div>
</div>
<button class="btn btn-submit" type="button">Upload File</button>
<input id="upload-button" type="file" name="uploads[]" multiple="multiple"></br>
```

TODO: finishing this article.