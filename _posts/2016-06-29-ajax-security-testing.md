---
layout: post
title: "AJAX Security Testing"
date: 2016-06-29 02:08:00
categories: JavaScript
featured_image: /images/cover.jpg
---

The next big project I have at [Sizzle](https://gosizzle.io) is to build a web scraper that parses company information from LinkedIn and dumps the information into the token. This effectively increases user productivity as the recruiter can focus more on job details instead of filling out company information. Because Sizzle runs on a LAMP stack and I wanted to scrape with Python, some glue between the two languages had to be created to successfully respond to the client-side AJAX requests.

## Setup of the Model

The model would mostly consist of an interaction between PHP and Python. On the client-side, jQuery would parse the input from the user consisting of a LinkedIn company URL, and this would be sent to the server-side in the form of an AJAX HTTP POST request. The AJAX endpoint was handled with PHP, which would then initiate the Python script containing the scraper. The web scraper would then parse the source code retrieved from LinkedIn with respect to a certain company, and pass this information back to PHP and ultimately back to the client.

<img src="http://i.imgur.com/vkDNjBc.png" width="60%">

This setup was easier said than done as the integration between PHP and Python got quite messy. There were a couple of "gluing" mechanisms I could use:

- **Bash:** This would be the easiest integration tool between the two languages because both had internal mechanisms to interact with the shell. The PHP `shell_exec` method could be used to initiate the Python script, which would simply dump information into `STDOUT`.

- **JSON:** JSON was a common format recognized by both languages and I could use this to my advantage. The company URL could be stored in a temporary JSON file, which would be read by Python and deleted to avoid clutter. Upon the completion of parsing, another JSON file would be generated with the data.

Ultimately, I decided to go with the second method after realizing how insecure the first one was. But, this website is all about reflecting on my mistakes and improving my software engineering skills, so I was obliged to write an article on it. I was never inclined to think that security was such a great hazard - probably because 1) I never had the skills to "hack" another website and 2) I was never in the position where some hacker hacked my program.

## AJAX and Security

AJAX's primary security flaw lies in its underlying model. AJAX can be thought of as a tube between the client-side and the server-side, where the client directly sends information to the server and the server responds accordingly. Of course, AJAX requests can be sent in the background, for example, weather applications refreshing every hour. However, there are times where the client sends information to the server through an AJAX request, like this instance.

The client has to be thought of as malicious and the server must protect against this. The flaw in the first gluing mechanism between PHP/Python was that the server was executing a shell script with direct input from the client. This essentially means that the client could send in malicious commands and get output easily because there was a direct tunnel from the client-side to the server-side.

## Testing it Out

In order to further explore the security vulnerabilities a web application could have with AJAX, I set up a simple application with Python, Flask, and jQuery. It is very minimal in functionality, but this is how it works:

- The client enters his or her name in an input field
- An AJAX POST request is sent to an endpoint on the server
- A script is executed and information is returned to the user

<img src="http://i.imgur.com/ADNJ8G6.png">

The security flaw this application will make is allowing the client access to the server-side by controlling what goes in the input field. The POST request the client sends will not be sanitized and because there is direct access to the shell, I will emulate scenarios where bad things will occur.

The front-end of the application is very simple. It contains an input field for the user and a button that will asynchronously send an HTTP POST request to the server with the help of jQuery.

```html
<div>
  <p>AJAX Security Testing</p>
  <input id="input" type="text" name="input">
  <button type="button" onclick="sendRequest()">Submit</button>
  <p id="output"></p>
</div>
```

After the data is successfully received from the server, the paragraph below the input field and button will populate with information. Exciting!

```javascript
function sendRequest() {
  var data = $("#input").val();
  $.ajax({
    method: 'POST',
    url: '/',
    data: {'data': $('#input').val()},
    dataType: 'json',
    success: function(data) {
      $('#output').text(data['output']);
    },
    error: function(xhr, status, error) {
      console.error(error);
    }
  });
}
```

As with any standard Flask application, here is what the directory structure of our project looks like:

```
test
|–– venv/
|–– templates/
    |–– index.html
|–– app.py
|–– test.py
```

`app.py` is the server - it will route the AJAX request, execute shell commands, and retrieve output from `test.py`, which is a variation of "Hello world."

```python
from flask import Flask, render_template, request, jsonify
import subprocess

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        resp = request.form["data"]
        command = "python3 test.py " + resp
        process = subprocess.Popen(command, stdout=subprocess.PIPE, shell=True)
        stdout = process.communicate()[0].decode("utf-8").strip()
        return jsonify({"output": stdout})
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

In the `index` method, we get the `data` variable from the JSON input, which is what the user enters on the client side. The `test.py` file (shown below) will be called and the output will go into the `stdout` variable.

```python
import sys

name = sys.argv[1]
print("Hello, {}!".format(name))
```

Finally, the server will return `{"output": stdout}` to the client, who will then see a sick message pop up in the paragraph tag. Upon entering "AJAX" in the input field, this is the result:

<img src="http://i.imgur.com/4hjp9bn.png">

This looks great and all, but the security vulnerability in the application has not been exploited. Nothing at all is sanitized in the `index` method, which means that whatever goes through the input field will be executed. With some very basic command line knowledge, namely the `ls` command, a hacker could find out which files and folders are stored in my directory by simply typing in `AJAX ; ls` into the input field:

<img src="http://i.imgur.com/kzdWQoj.png">

In fact, `ls` is just the tip of the iceberg. Because we're dealing with the shell directly, other commands like `cd` and `rm` work also. This means the hacker could change directories and even delete all of my data:

<img src="http://i.imgur.com/bn16vu1.png">

## Making AJAX More Secure

I'm not going to lie, making this sample project was a little scary because a small loophole like this could have devastating consequences for a large scale web application. Recursively deleting a directory takes seconds in comparison to the millions if not billions of dollars such a simple command costs to a company. However, there are some precautions developers can take to make AJAX requests more secure:

- **Avoid the command line:** The command line is an extremely powerful tool because it contains the blueprints to the entire computer if not network an application is running on. If an access point to a command line is not open, then it would be harder for a hacker to breach the system through malicious code entered through the client side.

- **Parse input on the front-end:** With the help of regular expressions or string matching, developers can block certain keywords from entering the server-side in the first place. If a blocked keyword was entered in an input field, then a message such as "Invalid input" could be flashed. The disadvantage to this approach is that because this happens on the front-end, a hacker could examine the code and find loopholes, making it less secure.

- **Parse input on the back-end:** This is definitely a big one. Languages such as PHP have methods like `escape_shell_cmd` which blocks special keywords from entering the server-side. Similar approaches are also used to stop [SQL injection](https://en.wikipedia.org/wiki/SQL_injection). The upside is that the server-side code is not visible, assuming the code is not open sourced.

- **Edit file permissions accordingly:** For the sake of a proof of concept, I changed the default web user (`www`) to have read and write permissions on my files. However, the `www` user should not have write permissions whatsoever and the read permissions should also be limited to a great extent. A basic rule of thumb should be to treat all clients as malicious and go forward from there.
