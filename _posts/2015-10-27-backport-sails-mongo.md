---
layout: post
title:  "Backporting Data on MongoDB in SailsJS"
date:   2015-10-27 9:00:00
author: "Matt Molnar"
categories: sailsjs mongodb
published: false
---
Many of us have had to do it. We implement changes to a live application that require modifying an existing/live database. I've just called this 'backporting' and up until just recently, I've only done it on SQL databases. Now that I've performed the same task in a NoSQL environment (specifically, MongoDB), I have gained some experience that has led me to rethink my choice of database type depending with respect to the development environment.

A little background first. I've come to really like SailsJS in the last half year. Prior Sails, I was still creating NodeJS backends and storing data in MongoDB. I have a fairly extensive RDBMS background, but Mongo just seemed to make sense. Store data in your database in the same format that you interact with it in the application. I started using embedded documents all over and really became fond of using Mongo with Node.

Fast forward to adding Sails to the mix. Sails comes with a database agnostic ORM called Waterline. Since I had been using MongoDB in the past, I decided to just stick with it for Node development. Though overall I would say I like Waterline, I was caught off guard when I discovered that it does not support embedded documents, regardless of the underlying database. So, I have been forced to normalize all my data structures again as if I'm using an RDBMS, even though I'm not.

I didn't care too much at first, just figured it was the cost of using a more structured environment. I did have to wonder though, why use a NoSQL database that supports embedded documents, if I have to interact with it as if it's a relational SQL database? If my advantages are gone, what's the point?

I kept pressing on though. Then comes my first instance of having to do a moderate data backport on a live Mongo/Sails/Node application. I had to add a data collection that would change the relationships of existing data records. In the past, with an RDBMS, I would write a SQL script to perform the backport and execute the script on the live database (after a backup and testing of course). This was my first solution idea here, just create a script to run directly on the Mongo database.

Well, remember how I said I have to normalize all my data even on Mongo? I now have to perform the equivalent of SQL join statements on a NoSQL database. Doing some research, Mongo doesn't support this very well. The common solution posted on QA sites like StackOverflow is that you should be using embedded documents instead. In the end, I wrote an imperative JS file to run at application boot to perform the backport. It took more time and finger-crossing than a simple transactional SQL script would have taken, but it thankfully worked.

**TL;DR** Using MongoDB with SailsJS/Waterline seems like mostly a bad idea on anything more than very simple applications or applications where the structure is likely to change moving forward. The big advantage of MongoDB is not having to normalize your data, if that feature is removed, then what's the point? I'll probably look into either using Mongo directly without Waterline, or switching over to a SQL database.
