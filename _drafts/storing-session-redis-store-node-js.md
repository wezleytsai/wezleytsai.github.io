---
layout: post
title: Storing Session Values using Redis Store in Node.js App
---
<!-- links -->

<!-- post -->

<!--excerpt-->

```javascript
var session = require('express-session');
var RedisStore = require('connect-redis')(session); // for sessions

app.use(session({
  store: new RedisStore({
    host: 'localhost',
    port: 6379,
  }),
  resave: false,
  saveUninitialized: false,
  secret: 'yoursecret'
}));
```
Need to run an instance of redis server:

```
>> brew install redis
>> redis-server
7778:M 27 Jul 22:28:06.462 # Server started, Redis version 3.0.2
7778:M 27 Jul 22:28:06.462 * The server is now ready to accept connections on port 6379
```

```javascript
req.session // can access session variables through here
```

Check if Redis is working using command line utility `redis-cli`.

```
>> redis-cli
127.0.0.1:6379> ping
PONG
```