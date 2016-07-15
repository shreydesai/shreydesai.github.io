---
layout: post
title: "MongoDB Connection Pooling"
date: 2016-07-14
---

In the midst of writing an API for [UT Courses](https://github.com/utexasdev/courses), I started suspecting some issues with MongoDB's speed. It was quite obvious that MongoDB couldn't be slow – at the end of the day, it __was__ a top-of-the-line NoSQL database meant for quick and efficient queries. This led me to suspect something was wrong in the way I set up the database or maybe the way I was using the database with my application. Upon reading the MongoDB documentation in depth, I was introduced to a new concept – connection pooling – which ended up solving the speed issue. Because this was the first database optimization I had made to date, an article surrounding this issue was obligatory.

## The Problem

[UT Courses](https://github.com/utexasdev/courses) is a search engine for all courses that [The University of Texas at Austin](https://utexas.edu) has to offer. UT Austin had a catalog dedicated to listing courses, but there were some problems with this approach:

- The courses were organized with each department, which made it easy for students to locate courses if they knew which department held the course. Otherwise, quite difficult.

- The official course catalog that allowed students to search through all courses was only open to UT Austin students and also incredibly slow.

With these issues in hand, I used [Node.js](https://nodejs.org) to scrape the department course catalogs and dump the information into MongoDB. Each course was represented in a JSON format:

```json
{
  "key": "C S 301K",
  "name": "Foundations of Logical Thought",
  "description": "Introductory logic in the context of computing; introduction to formal notations; basic proof techniques; sets, relations, and functions. Three lecture hours a week for one semester. Some sections also require one discussion hour a week; these are identified in the Course Schedule."
}
```

For the rest of the website, the idea would be that there would be an AJAX endpoint listening for `POST` requests. jQuery would be used on the front-end to make requests to the server and fetch the subsequent information from the database. The initial prototype of the endpoint, in retrospect, was disastrous.

```javascript
app.post('/key', function(req, res) {
  var query = req.body['search'];
  MongoClient.connect(uri, function(err, db) {
    var courses = db.collection('courses');
    var cursor = courses.find({'key': new RegExp(query, 'i')});
    cursor.toArray().then(function(docs) {
      res.json(docs);
    });
  });
});
```

There were a couple of efficiency problems with this. For starters, every time the server would receive a `POST` request, the function would attempt to 1) connect to the MongoDB database (which was hosted in the cloud, not `localhost:27017`) and 2) find the collection `courses` within the database.

Combined with the HTML rendering on the front-end, each request was taking around one to two seconds. Loaded queries would take even longer – maybe around four or five seconds. For non-programmers, this seems like an amazing result, but it really isn't. Imagine if Google took multiple seconds for each search query to their website. Scary, right?

## Connection Pooling

Upon a couple of reviews of the code, I came up with an idea – what if the connection to the MongoDB database was globally accessible and each `POST` request could just access this variable. Therefore, instead of connecting to the database **and** performing queries, the body of the function would just perform the query.

The MongoDB documentation called this concept **connection pooling**. The intent was the same as mine; multiple requests to query the database would use this one __global__ variable, which would drastically increase speed and efficiency.

Disgusted by my original prototype yet ecstatic with my new discoveries, it was time for some server-side code refactoring. This is what I came up with:

```javascript
var db = null,
    courses = null;

MongoClient.connect(uri, function(err, _db) {
  db = _db;
  courses = db.collection('courses');
  app.listen(3000, function() {
    console.log('Express server running on 127.0.0.1:3000');
  });
});

app.post('/key', function(req, res) {
  var query = req.body['search'];
  var cursor = courses.find({'key': new RegExp(query, 'i')});
  cursor.toArray().then(function(docs) {
    res.json(docs);
  });
});
```

This is how connection pooling worked with the application:

- Before the server started, MongoDB would attempt to connect with the database and `db` would contain a reference to `_db`, which was the actual database. This example assumes no errors, but there should be error handling realistically.

- Every time a `POST` request was made, the `courses` variable would readily provide an interface to perform the query. Until the server stopped running, `courses` could be re-used over and over again.

I decided to test out the concept by using `console.time` to see if my hunch was correct. Lo and behold, I was more than correct. What I found was that each query was only taking a millisecond tops while connecting to the database was taking a couple hundred milliseconds.

Of course, factoring in the response to the `POST` request and front-end rendering of the HTML, there would be a couple hundred milliseconds added to the overall operation, but connection pooling would save a ton of time.

I carried out a simple benchmark test after my little experiments; five departments at UT Austin were queried for a list of all of their courses. As expected, connection pooling made a huge difference in performance.

<div>
    <a href="https://plot.ly/~shreydesai/131/" target="_blank" title="" style="display: block; text-align: center;"><img src="https://plot.ly/~shreydesai/131.png" alt="" style="max-width: 100%;width: 1020px;"  width="1020" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="shreydesai:131"  src="https://plot.ly/embed.js" async></script>
</div>

<div>
    <a href="https://plot.ly/~shreydesai/129/" target="_blank" title="" style="display: block; text-align: center;"><img src="https://plot.ly/~shreydesai/129.png" alt="" style="max-width: 100%;width: 1020px;"  width="1020" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="shreydesai:129"  src="https://plot.ly/embed.js" async></script>
</div>
