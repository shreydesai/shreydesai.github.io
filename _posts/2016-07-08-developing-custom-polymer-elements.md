---
layout: post
title: "Developing Custom Polymer Elements"
date: 2016-07-08
---

Recently, I've had the opportunity to explore Polymer more in-depth. At [Sizzle](https://gosizzle.io), we use Polymer web components extensively; the combination of Polymer's ease of use and material design makes it an optimal library for UI development. Before I embarked on my next project with Polymer, I decided to delve deeper into building custom components. Polymer seems to provide a great developer interface for binding data and managing individual components. For experimentation purposes, I made a small app that performs `GET` requests to the [Glassdoor API](https://www.glassdoor.com/developer/index.htm) with AJAX, parses the JSON, and binds the data to a couple of elements.

## Polymer set up

Polymer's set up is quite simple. All you need to do is install Polymer and the necessary web components through Bower. Before making any installations, I suggest looking at the Polymer [element catalog](https://elements.polymer-project.org/), which has a massive library of web components and amazing documentation/demos as well. Because the library is quite extensive, you do not need to download all of the web components, so it is important to pick and choose.

Here is a list of Polymer web components I used for the project:

- Paper button

- Paper input

- Paper material

- Paper icon button

In order to set up Bower and install the components, fire up a Terminal shell and perform these commands:

```sh
$ bower update
$ bower init
$ bower install --save Polymer/polymer
$ bower install --save polymerelements/paper-button
$ bower install --save polymerelements/paper-input
$ bower install --save polymerelements/paper-material
$ bower install --save polymerelements/paper-icon-button
```

I ran into a bit of a problem with Bower – it was denying me access to some of my libraries because of administrator issues, and so the commands above had to be performed with `sudo`. If this happens, do `sudo bower install --allow-root --save <element>`.

Now, make a directory called `elements`. This will be used to store the custom Polymer components that we make. Also, an `index.html` is required in the root directory to run the application. Finally, because we will be making requests to the Glassdoor API and Polymer also makes requests, the application will need to run on a server, which can be done by this (assuming Python 3.x is installed):

```sh
$ python3 -m http.server 8000
```

The website will then server `index.html` at `0.0.0.0:8000`.

## Creating the home page

In order to start the development process, `index.html` will need to be written first. Polymer components are traditionally created inside of the `elements` directory and then aggregated/used on the home page. This application will be called `glassdoor-app`, so we can set this up in the `body` tag.

```html
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="bower_components/webcomponentsjs/webcomponents-lite.min.js"></script>
  <link href="elements/glassdoor-app.html" rel="import">
</head>
<body>
  <glassdoor-app></glassdoor-app>
</body>
</html>
```

Some notes about `index.html`:

- The `meta` tag is required for Polymer to work. Don't fight it.

- `webcomponents-lite.min.js` is a prerequisite file for Polymer. Polymer is a lightweight library built on the Web Components API so it requires an import statement at the very top.

- `elements/glassdoor-app.html` is where we will create the custom component, so that is why we include it as its own HTML tag in the `body` tag.

## Developing the component

Now that `index.html` is out of the way, we can start looking into `glassdoor-app.html` and begin developing the custom component. Polymer provides a template for getting started and it looks like this:

```html
<!DOCTYPE html>
<head>
</head>

<dom-module id="component_name">

  <style is="custom-style">
  </style>

  <template>
  </template>

</dom-module>

<script>
  Polymer({
    is: 'component_name',
    properties: {

    },
    ready: function(e) {

    }
  });
</script>
```

There are a couple of important sections in the Polymer custom component template that are worth looking at in detail:

- `head` - This is where all of the imports will go; this includes the Polymer library itself and other web components this component will be built upon.

- `style` - Styling the Polymer component is sometimes a pain because the web components don't cooperate sometimes, so custom styling is often required.

- `template` - The structure of the component will go here. For instance, if the component you're developing is a contact card that consists of a person's name and a star, these are two components that need to be set up.

- `script` - Because Polymer offers great data binding and event features, the logic and "back-end" functionality will be stored here. This is written in plain old JavaScript, so the learning curve is not steep at all.

Now, let's get to developing the component. Imports are the first thing we need to tackle, which is quite simple. All you need to do is import the HTML file that contains the element, which will probably be in the `bower_components` directory.

```html
<head>
  <link href="../bower_components/polymer/polymer.html" rel="import">
  <link href="../bower_components/paper-button/paper-button.html" rel="import">
  <link href="../bower_components/paper-input/paper-input.html" rel="import">
  <link href="../bower_components/paper-material/paper-material.html" rel="import">
  <link href="../bower_components/paper-icon-button/paper-icon-button.html" rel="import">
  <link href="../bower_components/font-roboto/roboto.html" rel="import">
</head>
```

The next thing we need to do is outline what the custom component will look like. In my case, there isn't much rocket science to be dealt with. I'll have an input field and submit button that triggers an asynchronous `GET` request to the Glassdoor API. Then, information pertaining to a particular company will be populated, such as: its Glassdoor image, name, industry, website, and overall rating.

<img src="http://i.imgur.com/QqMeMJZ.png" width="50%">

This will be set up in the `template` tag and it will look like this:

```html
<paper-material id="card" elevation="1">
  <div>
    <paper-input id="query" label="Enter company name"></paper-input>
    <paper-button on-click="submit">Submit</paper-button>
  </div>
  <p><b>Logo: </b></p>
  <paper-icon-button id="image" hidden={% raw %}{{loading}}{% endraw %} src={% raw %}{{image}}{% endraw %}></paper-icon-button>
  <p><b>Name: </b>{% raw %}{{name}}{% endraw %}</p>
  <p><b>Industry: </b>{% raw %}{{industry}}{% endraw %}</p>
  <p><b>Website: </b><a href={% raw %}{{website}}{% endraw %}>{% raw %}{{website}}{% endraw %}</a></p>
  <p><b>Rating: </b> {% raw %}{{rating}}{% endraw %}</p>
</paper-material>
```

One thing that might not seem so familiar is the {% raw %}`{{ }}`{% endraw %} syntax throughout the code. For those of you with prior [Jinja2](http://jinja.pocoo.org/) or [Mustache](https://mustache.github.io/) templating experience, this should not seem daunting at all. {% raw %}`{{ }}`{% endraw %} and `[[ ]]` are what Polymer uses for data binding, although I didn't use `[[ ]]` in this tutorial. Each set of braces has a variable name inside, which is considered a property; as expected, this will need to be logged inside the `properties` field in the `script` tag eventually.

Also, I was (grudgingly) forced to put in some custom styling.

```html
<style is="custom-style">
  * {
    font-family: "Roboto";
  }

  paper-material {
    padding: 2%;
  }

  paper-icon-button {
    height: 10%;
    width: 10%;
  }

  paper-input {
    display: inline-block;
  }

  paper-button {
    display: inline-block;
    background-color: #F7F7F7;
  }
</style>
```

The last, but most challenging, part is to put some functionality into the application. This will include creating the schema for the component's properties, requesting the API with AJAX, and binding the data to the component's elements once the request comes through. After some tinkering around with `iron-ajax`, the Polymer web component that provides support for AJAX requests, I decided not to pursue it and stick with jQuery instead. So, make sure to include the jQuery CDN before the Polymer script.

```html
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
```

The schema for the component's properties is trivial to set up. The `loading` property will determine whether the image has loaded, so the default value is `true` since we must presume the image isn't present. Next, the `image`, `name`, `industry`, `website`, and `rating` properties will all be `String`s because they represent text information.

```html
<script>
Polymer({
  is: 'glassdoor-app',
  properties: {
    loading: {
      type: Boolean,
      value: true
    },
    image: String,
    name: String,
    industry: String,
    website: String,
    rating: String
  }
});
</script>
```

Next, we need to write the functions required for the application to work. The first – and most important – function will be the one required to start the AJAX request. This will be called `submit`, because this is what we set the `on-click` property to be for the `paper-button`. The intent here is that the text from the input field will be passed as the company name for the API request and the appropriate response will come through.

```html
<script>
Polymer({
  is: 'glassdoor-app',
  properties: {
    loading: {
      type: Boolean,
      value: true
    },
    image: String,
    name: String,
    industry: String,
    website: String,
    rating: String
  },
  submit: function(e, detail) {
    var query = this.$.query.value;
    var _ = this;
    $.ajax({
      type: 'GET',
      url: 'http://api.glassdoor.com/api/api.htm',
      dataType: 'jsonp',
      data: {
        'v': 1,
        'format': 'json',
        't.p': 76164,
        't.k': 'cXzYgugsXrC',
        'userip': '24.6.216.224',
        'useragent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36',
        'action': 'employers',
        'q': query
      },
      success: function(d) {
        var company = d['response']['employers'][0];
        // pass information to components
      },
      error: function(x, s, e) {
        console.error(e);
      }
    });
  },
});
</script>
```

By default, `on-click` functions will have two parameters: `e` is the event itself and `detail` is some information regarding the event. In order to grab the value from the input, the `value` attribute is used. One important thing to note is that the `this.$` syntax is used to get the `id` attribute of the element we are targeting, namely the `paper-input` element.

Because we eventually need to 1) pass information to the components from the API response and 2) maintain scope, the `_` is used as a reference to `this`. The rest of the AJAX request is standard – the fields are what the Glassdoor API requires (more information can be found in their documentation). Finally, the data binding will occur in the `success` function.

In order to manipulate the data of the component's elements, some functions need to be set up in the Polymer `script` property. Here's what we should add:

```javascript
setImage: function(image) {
  this.image = image;
  this.loading = false;
},
setName: function(name) {
  this.name = name;
},
setIndustry: function(industry) {
  industry.length === 0 ?
    this.industry = 'NA' :
    this.industry = industry;
},
setWebsite: function(website) {
  this.website = website;
},
setRating: function(rating) {
  this.rating = rating;
}
```

I set up a ternary operator in `setIndustry` because some companies have not included what industry they belong to. Also, the `setImage` function will change the value of `loading` to `false`, allowing the image to display. The last thing we need to do is call these functions in the `success` function of the AJAX call so that the information is set.

```javascript
success: function(d) {
  var company = d['response']['employers'][0];
  _.setImage(company['squareLogo']);
  _.setName(company['name']);
  _.setIndustry(company['industry']);
  _.setWebsite(company['website']);
  _.setRating(company['ratingDescription']);
}
```

Great! That's all we need to do to finish off the Polymer component. Now, the `glassdoor-app.html` file should look something like this:

```html
<!DOCTYPE html>
<html>
<head>
  <link href="../bower_components/polymer/polymer.html" rel="import">
  <link href="../bower_components/paper-button/paper-button.html" rel="import">
  <link href="../bower_components/paper-input/paper-input.html" rel="import">
  <link href="../bower_components/paper-material/paper-material.html" rel="import">
  <link href="../bower_components/paper-icon-button/paper-icon-button.html" rel="import">
  <link href="../bower_components/font-roboto/roboto.html" rel="import">
</head>

<dom-module id="glassdoor-app">
  <style is="custom-style">
    * {
      font-family: "Roboto";
    }

    paper-material {
      padding: 2%;
    }

    paper-icon-button {
      height: 10%;
      width: 10%;
    }

    paper-input {
      display: inline-block;
    }

    paper-button {
      display: inline-block;
      background-color: #F7F7F7;
    }
  </style>

  <template>
    <paper-material id="card" elevation="1">
      <div>
        <paper-input id="query" label="Enter company name"></paper-input>
        <paper-button on-click="submit">Submit</paper-button>
      </div>
      <p><b>Logo: </b></p>
      <paper-icon-button id="image" hidden={{loading}} src={{image}}></paper-icon-button>
      <p><b>Name: </b>{{name}}</p>
      <p><b>Industry: </b>{{industry}}</p>
      <p><b>Website: </b><a href={{website}}>{{website}}</a></p>
      <p><b>Rating: </b> {{rating}}</p>
    </paper-material>
  </template>
</dom-module>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script>
Polymer({
  is: 'glassdoor-app',
  properties: {
    loading: {
      type: Boolean,
      value: true
    },
    image: String,
    name: String,
    industry: String,
    website: String,
    rating: String
  },
  submit: function(e, detail) {
    var query = this.$.query.value;
    var _ = this;
    $.ajax({
      type: 'GET',
      url: 'http://api.glassdoor.com/api/api.htm',
      dataType: 'jsonp',
      data: {
        'v': 1,
        'format': 'json',
        't.p': 76164,
        't.k': 'cXzYgugsXrC',
        'userip': '24.6.216.224',
        'useragent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36',
        'action': 'employers',
        'q': query
      },
      success: function(d) {
        var company = d['response']['employers'][0];
        _.setImage(company['squareLogo']);
        _.setName(company['name']);
        _.setIndustry(company['industry']);
        _.setWebsite(company['website']);
        _.setRating(company['ratingDescription']);
      },
      error: function(x, s, e) {
        console.error(e);
      }
    });
  },
  setImage: function(image) {
    this.image = image;
    this.loading = false;
  },
  setName: function(name) {
    this.name = name;
  },
  setIndustry: function(industry) {
    industry.length === 0 ?
      this.industry = 'NA' :
      this.industry = industry;
  },
  setWebsite: function(website) {
    this.website = website;
  },
  setRating: function(rating) {
    this.rating = rating;
  }
});
</script>

</html>
```

## Testing the component

Great! Now that the component is finished, we can test it out on the browser. The intended behavior is that a query – the company's name – will be entered and the submit button will trigger the AJAX request to the Glassdoor API. After some time, the information fields in the component should populate.

Make sure to start the server and head to `0.0.0.0:8000`.

```sh
$ python3 -m http.server 8000
$ open "0.0.0.0:8000"
```

Go ahead and enter a company's name in the input field. I put VMware, and sure enough, it worked. And, just to make sure the request wasn't a fluke, take a look at the Network section in the Chrome Developer Console and the HTTP request will show up.

<img src="http://i.imgur.com/6VYvOld.png" width="50%">
