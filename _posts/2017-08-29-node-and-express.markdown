---
layout: post
title:  "Node and Express"
date:   2017-08-29 14:59:30 -0700
permalink: /blog/:title
comments: true
---

This is a continuation of my effort to document my Node learning process. I am still going through [this](https://www.udemy.com/understand-nodejs/learn/v4/overview) excellent Udemy course and building upon very basic incremental examples to understand how and why Node can do the stuff that it does. In a [previous blog post](http://nataliesmith.ca/blog/node-servers) I wrote about how to create Node servers from scratch. This time, I’ll go through a few examples that use Node with Express to set up an app.

### What Is Express

Express is a framework to enhance Node and make Node easier to work with. The first example that I went through just created a very basic app using Express, which is shown below. I’ve commented nearly every single line of code in (maybe painful) detail just to really understand what’s happening at every stage. Important things to make note of:
- Express is simply a library providing functions and methods to make dealing with Node easier.
- Maybe the most important line is <span class="emphasis">var app = express();</span> as this is where we are creating our Express app.
- Maybe the second most important line is <span class="emphasis">app.listen(port)</span> as this is where the server is actually created.
- Anything that follows app.use is middleware. Middleware is something that happens between receiving a request and sending a response.
- Anything that follows app.get/post/otherHTTPverb is how we set up our app’s routes.

<script src="https://gist.github.com/natmegs/e0407b95e5cb86204d31759b633ec5d2.js"></script>
<script src="https://gist.github.com/natmegs/6693c1f12d98d8258b18103acd36151f.js"></script>

In this example, GET requests to the /form route will serve a static HTML file.

### Structuring the App

The next example, we look at a different way of structuring our app to separate routes and make them easier to deal with. Here, we have a folder called *controllers* in the same directory as app.js and form.htm, and controllers contains apiController.js and htmlController.js. The static file form.htm stays the same and everything that has changed from the previous example is indicated by comments.

<script src="https://gist.github.com/natmegs/ff9d6d2f2d6d32d00e1fcd4832e58cc7.js"></script>
<script src="https://gist.github.com/natmegs/4051e9179b7117b825c516269dbd7093.js"></script>
<script src="https://gist.github.com/natmegs/43b44d0735d0d05686cc9dd3638cd18f.js"></script>

### Using the Express Router

The next example deals with another way of structuring our app and uses the Router middleware provided by Express. Here, we have a folder called *routes* in the same directory as app.js, and routes contains index.js and users.js. Both of the routes files return an Express Router object, and then the main app.js file passes those Router objects to its own middleware. This effectively sets up routes with subroutes, where the routes defined in app.js are the main routes and the routes defined in the routes files are the subroutes that can be accessed via the main routes.

<script src="https://gist.github.com/natmegs/fda851a0b2d031248ac588ee7553c268.js"></script>
<script src="https://gist.github.com/natmegs/3beda9f4ccf51749e97f57dc4b7884f2.js"></script>
<script src="https://gist.github.com/natmegs/18e63953049cdbdfc0e346e5f390775d.js"></script>

These are all definitely just demonstrative examples that don’t do anything terribly exciting, but they helped me to understand how Express works with Node!
