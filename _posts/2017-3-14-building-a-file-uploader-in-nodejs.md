---
layout: post
title: Building a Progressive File Uploader in NodeJS
---

[<img src="{{ site.baseurl }}/images/uploader.png" alt="File Uploader" style="width: 400px;"/>]({{ site.baseurl }}/)

### Introduction
Uploading files is tricky (mostly conceptually), because it requires a bit of front-end, back-end and some inbetween layers. Building a system for uploading
files and *tracking the progress* is a bit harder. And finally, building a *resumable* uploader is quite a challenge. In this tutorial,
I will be building a progressive file uploader using **NodeJS** and **ExpressJS**. Later on, I will try to revist this and turn it into a resumable file uploader
using websockets. The concepts provided in this tutorial should easily translate to other languages and / or frameworks.

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

Lets navigate over to /public/css and make some modifications to the *styles.less* file here:

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

Now we need to write some Javascript. Inside of /public/js lets create an uploader.js file. 

Link this file in your /views/index.html as so:

```html
<script src="../js/uploader.js"></script>
```

Also, lets add a dependency (JQuery) which will be useful for use inside of uploader.js:

```html
<script
  src="https://code.jquery.com/jquery-2.2.4.min.js"
  integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44="
  crossorigin="anonymous"></script>
```

Now inside of /public/js/uploader.js we need to handle a few actions.

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

Next we need to register an event listener on the hidden input div, instead of listening to *onclick* this one will listen to 
*onchange*. The event listener on *upload-button-hidden* will only fire when a file is actually selected for upload, make it easy to
know when to begin firing HTTP requests to the back-end.

```javascript
$('#upload-button-hidden').on('change', function() {
    // upload logic goes here when a file is selected
}
```

Now inside of the callback function tied to *onchange* for *upload-button-hidden* we need to first
get the files indicated by the file upload form, and than append them to a form data object. 
Afterwards, we use JQuery to perform an AJAX POST request to our soon-to-be back-end:

```javascript
$('#upload-button-hidden').on('change', function() {

    // get files
    let files = $(this).get(0).files;
    
    // if any files exist
    if (files.length) {
    
        // new form data object
        let formData = new FormData();
        
        // fill form data with file data
        files.map((file) => {
            formData.append('uploads[]', file, file.name);
        });
        
        // send form data to server
        $.ajax({
            url: '/upload',
            type: 'POST',
            data: formData,
            context: this,
            processData: false,
            contentType: false,
            xhr: function () {
                let xhr = new XMLHttpRequest();
                
                // track progress
                xhr.upload.addEventListener('progress', function (event) {
                    if (event.lengthComputable) {
                        let percentComplete = event.loaded / event.total;
                        percentComplete = parseInt(percentComplete / 100);
                        
                        // update the progress bar
                        $('.progress-bar').width(percentComplete + '%');
                    }
                }, false);
            }
        });
    }
}
```

And that's about it for the minimal use case front-end. The CSS is going to be ugly,
but you can wrap it inside of a tile or style it in order to restrict the width. At the end of the
tutorial we will add some styles to make this uploader easier on the eyes.

### Building the Back-End

Our ExpressJS back-end now has to proccess the file data that is sent to /upload as the result of a 
POST request.

Inside of app.js, we need to add a hook for POST: /upload which will take a callback function
to process our data. To do this we need to do some pre-requisite plumbing:

1. Install the form data processing library "formidable"

```
npm install --save formidable
```
2. Require formidable, path and filesystem in your app.js

```javascript
let formidable = require('formidable');
let path = require('path');
let fs = require('fs');
```
3. Create a /uploads directory inside of your ExpressJS application for storing uploaded files

Now that we have our plumbing finished, inside of app.js lets create a 
hook for POST: /uploads:

```javascript
app.post('/upload', function(req, res, next) {
    let form = new formidable.IncomingForm();
    
    form.uploadDir = path.join(__dirname, '/uploads');
    
    form.on('file', function(field, title) {
        fs.rename(file.path, path.join(form.uploadDir, file.name));
    });
    
    form.on('error', function(err) {
        res.send({error: err});
    });
    
    form.on('end', function() {
        res.send('success');
    });
    
    form.parse(req);
});
```

And that's it for the back-end! 

There are a lot of things going on in this POST hook, for starters we 
are specifiying an upload directory and hooking into a listener for file uploads.
Inside of the file upload listener, we store the file in /uploads and provide it with a useful
file name.

Afterwards, we have hooks to catch errors, or a successful upload. Ours 
aren't very robust, but eventually they should probably be extended to provide more valuable information 
to the client.

If you run your application now, you should be able to upload files at http://localhost:3000 and than
view the uploaded files inside of the /uploads directory in your application code.

### Styling the Uploader

For simplicities sake, I tried not to add too much CSS to the tutorial itself. But styling the uploader is actually quite straitforwards.
You will want a fixed width container for the uploader, and some type of background for the progress bar. You can see an example I mocked up in
[JSBin](http://jsbin.com/fujoqawoba/edit?html,css,output). Which contains this CSS:

```css
.tile {
  display: block;
  width: 400px;
  height: 200px;
  background-color: #fff;
  padding: 15px;
}

.tile__title {
  text-align: center;
  font-weight: bold;
}

.tile__contents {
  margin-top: 25px;
}

.progress {
  width: 100%;
  height: 25px;
  border: 1px solid #333;
  border-radius: 2px;
}

.progress-bar {
  width: 50%;
  height: 25px;
  background: repeating-linear-gradient(
  -55deg,
  blue,
  blue 10px,
  orange 10px,
  orange 20px
);
  
}

.progress-bar__text {
  line-height: 25px;
  text-align: center;
}

.button {
  background-color: orange;
  color: #fff;
  width: 100px;
  height: 25px;
  margin-top: 40px;
  text-align: center;
  line-height: 25px;
  border-radius: 2px;
  margin-left: auto;
  margin-right: auto;
  padding: 10px;
  cursor: pointer;
}
```
And produces the screenshot at the beginning of the tutorial.