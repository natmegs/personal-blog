---
layout: post
title:  "SQL Server and Node with Sequelize"
date:   2018-05-01 15:30:30 -0700
permalink: /blog/:title
comments: true
---

I recently was working on a project that required interacting with a MSSQL database through a Node application. I waded through the depths of the internet to piece together my solution, taking insights from a variety of published solutions that didn’t quite meet my requirements. After some cycles of frustration/exhaustion, my Node API is now talking to my MSSQL server happily, and I can lay out the solution I came to with explanations for my future self (and anyone else who may need to do something similar).

###### Preface

Before I get into the implementation details, it’s probably important to recognize what some of my constraints were going into the project. First, the goal of (this part of) the project was to use Node to connect to a SQL Server database, create a users table, and perform basic lookups and insertions to lay the foundation for user authentication. Before I knew anything about how to accomplish my goal of interacting with my database using Node, I knew that my base tech stack was going to look like this:

- Node: web API backend
- Express: web framework for Node
- Passport: authentication middleware for Node
- MSSQL: database server

These constraints impacted the choices I made with respect to additional libraries and packages. Node was chosen due to my familiarity with Javascript, and Express and Passport were chosen since I have basic experience with those packages. MSSQL as the database layer was chosen by default (not by me).


###### Interacting with SQL in Node

It became clear very quickly that SQL and Node are not used together nearly as often as relational databases such as MongoDB with Node. The documentation/tutorial situation on the topic was thus disparate and generally lacking. There are several packages published on npm with the goal of connecting and interacting with a SQL server and database in Node. 

- [sql](https://www.npmjs.com/package/sql)
- [node-sql-template](https://www.npmjs.com/package/node-sql-template) 
- [node-sql](https://www.npmjs.com/package/node-sql) 
- [nodesql](https://www.npmjs.com/package/nodesql) 
- [mssql](https://www.npmjs.com/package/mssql) 
- [tedious](https://www.npmjs.com/package/tedious) 
- [express4-tedious](https://www.npmjs.com/package/express4-tedious) 
- [sequelize](https://www.npmjs.com/package/sequelize) 

I will make it clear that my main priority for choosing libraries was documentation. It would not have been a good use of my time, in this particular project, to have to read library source code to understand how to accomplish basic tasks. After being led down the google search rabbit hole for this list, I ended up landing on <span class="code">tedious</span>, since it is officially supported by Microsoft and seemed to have decent documentation. I ended up finding <span class="code">express4-tedious</span> through one of the Microsoft/Azure code samples, and this seemed promising but had no documentation past a very basic use case. In browsing github issues for tedious, I came across some [survey results](https://github.com/tediousjs/tedious/issues/690) that involved input from users who use Node to interact with SQL server (bingo). Despite the fact that there were a relatively small amount of total responses and the responder audience was clearly skewed by those who were already exposed to tedious in some sense, it was helpful to see how others are solving similar problems. 

The results showed that tedious was the most popular driver used to connect Node apps to SQL server, but I noticed some responses that indicated that responders were using tedious via <span class="code">sequelize</span>. Sequelize is a promise-based ORM for Node that supports mssql (among others), and its extensive documentation and even some blog tutorials were enough to convince me.

It is important to note that sequelize is intended for use WITH a mssql driver, so tedious and sequelize are used together:

<span class="code">npm install tedious sequelize</span>


###### Connecting to a SQL Server and Database 

Ok, I still didn’t have much other than a starting point with tedious and sequelize. To begin with, I needed to create a connection to the SQL server. First I created a new Sequelize instance in my main app.js Node file and tested the connection using the sequelize authenticate method:

<script src="https://gist.github.com/natmegs/cdc78f2dd0d989594ab69f7664bdaedb.js"></script>

Cool, now I know I can connect to the database. Next step is to do something useful with it…


###### Creating Models

Sequelize has the concept of database models, where we can define the shape of the data we are going to store in our database. The models correspond to tables in the database. In my case, I knew I needed a user table so I created a models folder in my project and added a user.js file to this folder. Here I defined my User model (and therefore my user table).

<script src="https://gist.github.com/natmegs/38b32b8e0defe9975d8f99ff5814f6d8.js"></script>

Since I will likely have other models as well as User eventually, I need a place to bring together all the models in one place. In the models folder, I made an index.js file to do exactly this.

<script src="https://gist.github.com/natmegs/23352adb72127f1a56dbe33ff4539263.js"></script>

I use the sequelize define method as well as sequelize types to create our user definition. The entries (email, password, etc) will correspond to columns in my user table in my database. Then I load all the models in the models folder into a single exported object in models/index.js.

###### Database Synchronization

So now I have

- Two installed packages (sequelize, tedious)
- A test connection using sequelize
- A user model

But if I look at my database it’s still empty, so I need to create the user table in the database. Since I have my model structure, I can hand off the work to sequelize using the <span class="code">sync</span> method. This method will sync (create tables for) all models that aren’t already in the database.

In my case, I’m using Express (specifically the Express generator) to set up my Node application so the best place to sync the database is before the server starts listening to its designated port. With the Express generator, this happens in the bin/www file:

<script src="https://gist.github.com/natmegs/5a63c601689a806eec12448283d04fe0.js"></script>

Now, I can interact with my database and tables through the models export from anywhere in the Node app.

###### Querying the Database

Now that I have my models set up and my database synchronized, I can query the database tables anywhere in the app by importing models. For example:

<script src="https://gist.github.com/natmegs/f16e24bc5813a5db57e0c864c9952610.js"></script>

The methods <span class="code">findOne</span> and <span class="code">create</span> are both provided by the Sequelize library, and return model instances.

I will do a follow up post with more details on implementing authentication using Node, Express, Passport, Sequelize, and MSSQL. But for now, this is the basic gist of how to get going with connecting to, setting up tables in, and querying a database.



