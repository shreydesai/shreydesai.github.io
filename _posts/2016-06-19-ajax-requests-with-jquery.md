---
layout: post
title: "AJAX Requests with jQuery"
date: 2016-06-19 11:54:00
categories: JavaScript
featured_image: /images/cover.jpg
---

This week at [Sizzle](http://gosizzle.io), I moved on from fixing front-end bugs to developing some new features on the back-end of the product. Sizzle currently develops "tokens" for recruiters, which are personalized bulletins with information for a particular role. These tokens are both informative and engaging, making them an optimal tool to attract prospective candidates. One of the issues with the token, though, is that the data must be manually inputted. My task this week was to develop a feature that would auto-populate company information from LinkedIn.

## HTTP Requests

I had done some previous work with web scraping before; my most recent [project](https://github.com/shreydesai/ap-scores) was getting the 2015 AP Scores from a website with Python. The concept would be similar for this feature as well. Each company that sets up a page on LinkedIn can be identified with a particular URL, for example, [https://linkedin.com/company/amazon](https://linkedin.com/company/amazon) for Amazon. The company page contains a lot of information, such as the company's name, description, and some images. My job was to parse this information and upload it to the token accordingly.

Because the scraping jobs had to be handled on a case-by-case basis, I had to learn more about requests and servers. Servers are essentially computers that run on an infinite loop and handle requests made by a client. In this case, the client would be the recruiter requesting information from LinkedIn. The feature I had to develop would handle this request accordingly and give the information back to the client. Most requests made to servers are `GET` requests, which simply ask the server to "get" the information on a page. If you do `curl google.com` in a Terminal window, the console will print out Google's HTML.

The request I would get from the client would not be a `GET` request because the client would have to give me some information - the URL of their LinkedIn company page. In order for the information to come with the request, the server would need to handle a `POST` request. `POST` requests are usually sent by HTML forms and these types of requests send data to the server only once. Now that the type of request was resolved, jQuery would be used to make the request.

## Introduction to AJAX

Requests nowadays are commonly sent through AJAX, which stands for Asynchronous JavaScript and XML. However, a lot of data is sent through JSON these days as a replacement for XML. AJAX should probably be called AJAJ, but I think the former sounds much slicker. Regardless, AJAX is very valuable:

- It is asynchronous in nature, which means that the queue of requests to the server is not interrupted and blocked by a new one, namely the one sent by the client. Even though the `async` parameter in the jQuery call could be set to `false`, I'm pretty sure this is deprecated now.

- It does not require a page refresh for new information to populate because the server handles the AJAX request without making a `GET` request. This makes new information on a website seem "real time." For example, if you visit Google Finance in the middle of the day and click on a company's stock, it will update in real time as a result of AJAX requests to some server.

For the actual jQuery code, `ajax` is a function that we need to declare. It takes some parameters in a JSON format, such as `type`, `dataType`, `data`, `url`, `success`, and `error`. The documentation covers much more variables, but usually not all of them are necessary. This is what an AJAX call would look like for a client requesting Amazon's information.

```javascript
$.ajax({
  type: 'POST',
  dataType: 'json',
  data: {link: "https://linkedin.com/company/amazon"},
  url: '/scraper',
  success: function(data) {
    console.log(data);
  },
  error: function(xhr, status, error) {
    console.log(error);
  }
});
```

As an overview, this particular AJAX request was a `POST` request. The data transferred to and from the server was in a JSON format, which is why I declared the `dataType` key. Next, because a `POST` request carries data with it, the `data` variable contained a JSON object with information, namely the company's LinkedIn URL. `url` specifies which server route is going to handle the request, which I will describe more in detail below. Lastly, `success` and `error` are functions that handle the response from the server.

## Implementation of AJAX

Requests are a two-way transfer between the client and the server. The client sends the request and the server must respond to the request in an equitable manner. That is why the AJAX function has `success` and `error` attributes. Below is an example with [Flask](flask.pocoo.org) of how the AJAX call above could be handled.

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/scraper")
def scraper():
  """Handles scraping jobs from the client"""
  if request.method == "POST":
    data = request.args.get("data")
    url = data["link"]
    company_info = parse_linkedin(url)
    return jsonify(company_info)

def parse_linkedin(url):
  """Scrapes LinkedIn for information"""
  pass

if __name__ == "__main__":
  app.run()
```

If we assume this is the server that the AJAX request is sent to, the `scraper` function will handle the `POST` request by calling the `parse_linkedin` function. I omitted the code in `parse_linkedin` for brevity purposes, but this function would return something and this would be passed back to the AJAX function in the form of `success` or `error` depending on how the server handled the request. Another important thing to note is that I used the `jsonify` function because it is imperative that the data going in and out of the server is in the same format, which is JSON for our purposes. Most programming languages support the JSON data format through their various data structures, so it is possible to handle data across multiple languages.

Now that I learned more about requests, the next step I want to take is delve deeper into the architecture of REST and learn about other HTTP methods such as   `PUT` and `DELETE`. My goal is to create a sample application in Flask that has a RESTful API and I'll test it out with `curl` and post (or should I say `POST`) about it.
