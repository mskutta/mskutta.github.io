---
layout: post
title: "Sitecore How-To: Helpful Mongo Queries for Sitecore xDB"
date:   2018-09-21 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Mike Skutta, Peter Amundson
excerpt: These are a few helpful queries for Sitecore's xDB Mongo database to retrieve information about the contacts and visits.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

These are a few helpful queries for Sitecore's xDB Mongo database to retrieve information about the contacts and visits.

### Get Unique URL Counts

This reviews all of the site interactions and retrieves the unique pages (query strings are ignored) and the number of unique interactions.

```javascript
var cursor = db.getCollection('Interactions').aggregate([
{
  $project:
  {
    "Pages.Url.Path": 1,
    _id: 0
  }
},
{
  $unwind: "$Pages"
},
{
  $group:
  {
    "_id": "$Pages.Url.Path",
    "count": { $sum: 1 }
  }
},
{
  $sort: { "count": -1 }
}]);

while (cursor.hasNext()) {
  print(cursor.next());
}
```

### Get Unique URL and Querystring Counts

This reviews all of the site interactions and retrieves the unique pages (query strings are unique) and the number of unique interactions.

```javascript
db.getCollection('Interactions').aggregate([ 
{
  $project:
  {
    "Pages.Url.QueryString": 1,
    "Pages.Url.Path": 1,
    _id:0
  }
},
{
  $unwind : "$Pages" 
},
{
  $group:
  {
    "_id":"$Pages.Url.QueryString",
    "url": {$addToSet:"$Pages.Url.Path"},
    "count":{$sum:1}
  }
},
{
  $sort: {"count":-1}
}],
{ allowDiskUse: true });
```

### Get Unique User Agent Counts

This reviews all of the site interactions and retrieves the unique user agents and the number of unique interactions.

```javascript
var cursor = db.getCollection('Interactions').aggregate([ 
{ $project:
  {
    "UserAgent": 1,
    _id:0
  }
},
{
  $group:
  {
    "_id":"$UserAgent", 
    "count":{$sum:1}
  }
},
{
  $sort: {"count":-1}
}]);

while (cursor.hasNext()) {
  print(cursor.next());
}
```

### Get Unique User Agent by Unique Page Counts

This reviews all of the site interactions and retrieves the unique user agents based on the number of pages that the user agent has interacted with.

```javascript
var cursor = db.getCollection('Interactions').aggregate([ 
{
  $project:
  {
    "UserAgent": 1,
    "PageCount": { $size: "$Pages" }, 
    _id:0
  }
},
{
  $match: {
    "PageCount": { "$eq": 1 }
  }
},
{
  $group:
  {
    "_id":"$UserAgent", 
    "count":{$sum:1}
  }
},
{
  $sort: {"count":-1}
}]);

while (cursor.hasNext()) {
  print(cursor.next());
}
```

### Get Number of Pages by Visitor Session

This reviews all of the site interactions and retrieves the number of pages per visitor session.

```javascript
var cursor = db.getCollection('Interactions').aggregate([
{
  $project: 
  {
    _id:1,
    UserAgent:1,
    Pages: { $size: "$Pages" } 
  }
},
{
  $sort: {"Pages":-1}
}],
{
  allowDiskUse: true 
});

while (cursor.hasNext()) {
  print(cursor.next());
}
```