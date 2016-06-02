---
layout: post
title: "Locally Setting Up the Facebook SDK"
date: 2016-05-27 07:40:00
categories: API
featured_image: /images/cover.jpg
---

Since its conception, the Facebook SDK has been notorious for being difficult to set up. In comparison to other APIs, Facebook has done a rather poor job making its web services accessible. The learning curve is steep, the documentation is vague, and many elements are hard to use. Personally, I feel like other SDKs/APIs such as Twitter and Soundcloud have been much simpler; maybe this is in part because third-party developers have created wrapper APIs to access the services, but the documentation and support is better overall. 

Nevertheless, I finally figured how to set up the Facebook SDK on a local server with Node.js and Express. This should work with any web server but because this process is completely undocumented, I'll go through the process. 

## Registering the app

Head over to [https://developers.facebook.com/apps](https://developers.facebook.com/apps) to get started. Click on *Create a New App*, which should open a modal window where the name of the app needs to be inputted. Next is the important step - when registering the app on the WWW platform, put `localhost` as the *Site URL* box. This is because the app will be run on your computer and subsequent (asychronous) requests will be made to load the Facebook SDK.

## Writing some code

The app itself is bland because all it does is demonstrate that the Facebook SDK actually words. Create an `index.html` file and use a text editor to edit its contents. Designate a space inside the `body` tags to store the code for the SDK.


```html
<html>
    <body>

    <!-- Facebook SDK goes here -->

    </body>
</html>
```

On the same page that had the *Site URL* box, there should be an area that contains a pre-written script to load the Facebook SDK with jQuery. Copy and paste that code bit into the HTML. Lastly, add the test code that Facebook provides under the script where you initially put the SDK. If you put it above the SDK, you'll probably run into some "not-found" JavaScript console errors.

```html
<html>
    <body>

        <!-- Load Facebook SDK -->
        <script>
          window.fbAsyncInit = function() {
            FB.init({
              appId      : '1340412182640737',
              xfbml      : true,
              version    : 'v2.6'
            });
          };

          (function(d, s, id){
             var js, fjs = d.getElementsByTagName(s)[0];
             if (d.getElementById(id)) {return;}
             js = d.createElement(s); js.id = id;
             js.src = "//connect.facebook.net/en_US/sdk.js";
             fjs.parentNode.insertBefore(js, fjs);
           }(document, 'script', 'facebook-jssdk'));
        </script>

        <!-- Facebook SDK Test code -->
        <div
          class="fb-like"
          data-share="true"
          data-width="450"
          data-show-faces="true">
        </div>

    </body>
</html>
```

## Starting the server

This should work with any server, but since I was going to make a Node.js app, I'm using Express. Create an `app.js` file that serves the `index.html` we created above.

```javascript
var express = require('express');
var app = express();

app.get('/', function(req, res) {
    res.sendFile('index.html', {root: __dirname});
});

app.listen(3000, function() {
    console.log('Server started on port 3000');
});
```

Fire up a shell and use `cd` to navigate to the directory where the project files are stored. `node app.js` should start the server and the website will be served at `localhost:3000` or `127.0.0.1:3000`. Hopefully, the website should display a like box and the JavaScript console should show no errors.
