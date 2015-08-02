---
layout: post
title: MongoDB Setup - Data Directory Not Found or Permission Denied
---

<!-- links -->
[mongodb docs]: http://docs.mongodb.org/manual/installation/

<!-- post -->
Fixing error messages that I got when setting up MongoDB.

<!--excerpt-->

From the command line, install MongoDB using `brew`. Or install it manually following the MongoDB [manual] [mongodb docs].

```bash
>> brew install mongodb
```

mongod is the daemon process for running the mongoDB server. Run `mongod` in the command line to get it started. When I did that I got the following error:

```bash
>> mongod
2015-07-26T16:15:38.473-0700 I STORAGE  [initandlisten] exception in initAndListen: 29 Data directory /data/db not found., terminating
2015-07-26T16:15:38.473-0700 I CONTROL  [initandlisten] dbexit:  rc: 100
```

This is because mongoDB saves data into the directory /data/db by default so it needs to be created (you can also specify another director by using the `--dbpath` option). Use `mkdir` and you might have to `sudo` it if permission is denied.

```bash
>> sudo mkdir /data/db
```

I tried `mongod` again and got another error:

```bash
>> mongod
2015-07-26T16:17:57.653-0700 I STORAGE  [initandlisten] exception in initAndListen: 98 Unable to create/open lock file: /data/db/mongod.lock errno:13 Permission denied Is a mongod instance already running?, terminating
2015-07-26T16:17:57.654-0700 I CONTROL  [initandlisten] dbexit:  rc: 100
```

This was due to permission issues to the /data/db directory.

```bash
>> sudo chown -R $USER /data/db
```

That finally resolved all the issues. With mongod running, I can start the mongo shell by entering `mongo` (on a new shell tab) to connect to my running MongoDB instance.

```bash
>> mongo
> show dbs
```
