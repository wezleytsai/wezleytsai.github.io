---
layout: post
title: MongoDB Connection Pooling with Node.js Modules
---

<!-- links -->
[mongo]: https://mongodb.github.io/node-mongodb-native/driver-articles/mongoclient.html

<!-- post -->

How to implement MongoClient connection pooling in a simple Node.js/MongoDB web app.

<!--excerpt-->

## Not So Good ##
Here is a sample node.js app. On a `GET` request, the app will do something with the database (maybe fetch some data). The database operations are confined in the `database.js` module, which is great, but a connection with the MongoDB is opened/closed with each operation, which can be costly and inefficient.

**app.js**

```javascript
var express = require('express');
var app = express();
var database = require('./database.js');

app.listen(3000);
console.log("Listening on port 3000");

app.get("/", function(req, res) {
  //  do some stuff
  database.doSomethingWithDatabase(someCallback);
});
```

**database.js**

```javascript
var mongodb = require('mongodb');
var MongoClient = mongodb.MongoClient;
var mongoUrl = 'mongodb://127.0.0.1:27017/db_name';

exports.doSomethingWithDatabase = function(callback){
  MongoClient.connect(mongoUrl, function(err, db) {
    if( err ) throw err;
    // connected to db
    db.collection('db_name').find({}, function(err, docs) {
      // do something
      callback(docs);
      // close connection to db
      db.close();
    });
  });
};
```

## Better ##
This is the [example] [mongo] on the mongodb github on **connection pooling**. It's a great and simple example, but it requires that database operations be within the same module that `MongoClient.connect` is in to have access to the `db` variable.

**app.js**

```javascript
var express = require('express');
var app = express();

var mongodb = require('mongodb');
var MongoClient = mongodb.MongoClient;
var mongoUrl = 'mongodb://127.0.0.1:27017/db_name';
var db;

// Initialize connection once
MongoClient.connect(mongoUrl, function(err, database) {
  if(err) throw err;

  db = database;

  // Start the application after the database connection is ready
  app.listen(3000);
  console.log("Listening on port 3000");
});

// Reuse database object in request handlers
app.get("/", function(req, res) {
  db.collection('db_name').find({}, function(err, docs) {
    // docs found using the same db connection
    // do something
  });
});
```

## Best ##
On bigger apps, you're probably separating your app into several modules by logic. This is the best way to isolate database operations in a `database` module.

**app.js**

```javascript
var express = require('express');
var app = express();
var database = require('./database.js');

// Initialize connection
database.connect(function() {
  // Start the application after the database connection is ready
  app.listen(3000);
  console.log("Listening on port 3000");
  
  app.get("/", function(req, res) {
    //  do some stuff
    database.doSomethingWithDatabase(someCallback);
  });
});

```

**database.js**

```javascript
var mongodb = require('mongodb');
var MongoClient = mongodb.MongoClient;
var mongoUrl = 'mongodb://127.0.0.1:27017/db_name';
var db;

exports.connect = function(callback) {
  MongoClient.connect(mongoUrl, function(err, database) {
    if( err ) throw err;
    db = database;
    callback();
  }
}

exports.doSomethingWithDatabase = function(callback){
  // this is using the same db connection
  db.collection('db_name').find({}, function(err, docs) {
    // do something
    callback(docs);
  });
};
```