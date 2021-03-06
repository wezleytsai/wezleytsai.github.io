---
layout: post
title: Saving Bcrypt Hashed Passwords to Database with Bookshelf.js
---

<!-- links -->
[trigger-then]: https://github.com/tgriesser/bookshelf/issues/252
[bcrypt-npm]: https://www.npmjs.com/package/bcrypt
[bcrypt-wiki]: https://en.wikipedia.org/wiki/Bcrypt
[dict-attack-wiki]: https://en.wikipedia.org/wiki/Dictionary_attack

<!-- post -->
The slow-by-design processing time is a strength of the bcrypt hashing algorithm, but it also means that it needs to be performed asynchronously so as not to block. I ran into some issues when trying to save hashed passwords into the databse using the Bookshelf.js ORM - here is how I resolved them.

<!--excerpt-->

## Problem ##

Here is a `User` model representing data to be stored in a "users" table. I have a listener for the `"creating"` event, which is triggered before the model is being inserted. The event handler is intended to hash the password so the user's password is never directly saved into the database.

```javascript
var bcrypt = require('bcrypt');

var User = bookshelf.Model.extend({
  tableName: 'users',
  initialize: function() {
    this.on('creating', this.hashPassword, this);
  },
  hashPassword: function(model, attrs, options) {
    bcrypt.hash(model.attributes.password, 10, function(err, hash) {
      if( err ) throw err;
      model.set('password', hash);
    });
  }
});
```

The problem I had was that the model was being saved to the database before the async `bcrypt.hash` invoked the callback. As a result, this was what I got during testing:

```javascript
new User({username: 'dayman', password: 'nightman'})
.save() 
.then(function(model) {
  console.log(model); // password = '$2a$10$YDc3uqPpMo9yKTfQDL2DAeTH6Hme2w1GZTI2bl0qxmp3vptm99Ax2'
  new User({username: 'dayman'})
  .fetch()
  .then(function(found) {
    console.log(found); // password = 'nightman'
  });
});

```

The model had its `password` property set to the result of the hash as expected, but the original password "nightman" had already been saved into the database. One option is to `save` the model again after the first `console.log`. This would update the password in the database to the hash, but the original password would have been momentarily saved in the database, which is a security risk.

## Solution ##

After some research, I found this [discussion] [trigger-then] on github and found out that the lifecycle events of Bookshelf, such as `"creating"` and `"saving"`, are _promise-aware_, in that if the event handlers returned **promises**, the events will wait for the promise to be resolved or rejected before progressing. If the promise is rejected, the save process will be interrupted.

So the solution is to wrap the async process in a promise like so:

```javascript
var bcrypt = require('bcrypt');
var Promise  = require('bluebird');

var User = bookshelf.Model.extend({
  tableName: 'users',
  initialize: function() {
    this.on('creating', this.hashPassword, this);
  },
  hashPassword: function(model, attrs, options) {
    return new Promise(function(resolve, reject) {
      bcrypt.hash(model.attributes.password, 10, function(err, hash) {
        if( err ) reject(err);
        model.set('password', hash);
        resolve(hash); // data is created only after this occurs
      });
    });
  }
});
```

And the password saved into the database is indeed the 60 character bcrypt hash:

```javascript
// delete the old 'dayman' user, and re-insert:
new User({username: 'dayman', password: 'nightman'})
.save()
.then(function(model) {
  console.log(model); // password = '$2a$10$5md31uoYdFqmJSFVJ/SH1ewm4idtiTk4sX8tZwo0ZVGwVE4QAGwyC'
  new User({username: 'dayman'})
  .fetch()
  .then(function(found) {
    console.log(found); // password = '$2a$10$5md31uoYdFqmJSFVJ/SH1ewm4idtiTk4sX8tZwo0ZVGwVE4QAGwyC'
  });
});

```

