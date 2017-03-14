---
layout: post
title: Building a Progressive File Uploader in ExpressJS
---

> This blog post is on version: 0.2. You can read it right now, but it is not yet finished. 

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


### Building the Front-End 
If we browse through the files created by the ExpressJS generator, we will notice a few folders in the heirarchy. Right now we are 
interested in the index file under "views". This is the default view which will be loaded when a user visits www.your-url.com.

Inside of this file, delete the markup and replace it with:

```html
<h2>File Uploader</h2>
<div class="progress">
    <div class="progress-bar" role="progressbar"></div>
</div>
<button class="btn btn-submit upload-button" type="button">Upload File</button>
<input id="upload-button-hidden" type="file" name="uploads[]" multiple="multiple"></br>
```

The markup probably isn't pretty (yet), but serves some important functions. First off, it gives a submit button as a button class
which we can style - and an input button which we will hide and call when the styled button is clicked. This means we can have both
*choose file* and *upload file* from the same button markup. Confused? Keep reading, it will make sense soon.

Lets navigate over to /public/css and make some modifications to the *styles.less* file here. We only need one style right now:

```css
#upload-button { display: none }
```

Again, we don't want to have a button for file selection *and* file upload. So the input will serve as file selection as far as the
markup is concerned but will not render inside of the page.

Also, we need to add some styles so our progress bar will display on the screen:

```css
.progress {
    height: 25px;
    width: 100%;
    border 1px solid #333;
}
.progress-bar {
    height: 25px;
    background-color: blue;
    width: 0%;
}
```

Now we need to write some Javascript. Inside of /public/javascripts lets create an uploader.js file. 

Link this file in your /views/index.html as so:

```html
<script src="javascripts/uploader.js"></script>
```

Also, lets add a dependency (JQuery) which will be useful for use inside of uploader.js:

```html
<script
  src="https://code.jquery.com/jquery-2.2.4.min.js"
  integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44="
  crossorigin="anonymous"></script>
```

Now inside of /public/javascripts/uploader.js we need to handle a few actions.

First off, we need to make sure that the width of our progress bar is always set to 0% before uploading:

```javascript
$('.upload-button').on('click', function (){
    $('#upload-button-hidden').click();
    $('.progress-bar').width('0%');
});
```

This Javascript code will register an event listener on the upload-button class which will do the following when a click is registered:
1. Click on the hidden upload button - opening a file input dialog
2. Set the width of the progress bar to 0%, indicating that the file upload has not yet started


