---
layout: post
date: '2012-07-22T14:50:00+02:00'
title: 'Improve Redis performance: simultaneous max clients'
description: 'How to increase Redis maximum clients number'
comments: true
tags:
- 'max number of clients reached'
- 'redis'
- 'redis performance'
- 'redis maxclients'
- 'ulimit'
tumblr_url: http://yadeb.tumblr.com/post/27775570494/improve-redis-performance-simultaneous-max
---
One day at work, we had to handle a DDoS attack. While I was investigating the problem, I saw that our Redis server was refusing new connections saying “max number of clients reached”.

Of course, this was due to the huge amount of requests received (200K / sec) but Redis could handle only 1024 simultaneous clients compared to what’s in the documentation (10000). The problem was not Redis but our server configuration.

This is the kind of problem where you will not find a lot of documentation on Google because this is happening only in rare conditions, often when a lot of traffic and hardware resources are required.

Here is the simple fix:

> NOTE: Code snippets are from an Ubuntu server and it may differ on your own.

First, check `maxclients` variable in your Redis configuration file and set a limit or comment this line:

```
# /etc/redis.conf
# maxclients 128
```

Redis need a file descriptor for each client and it does that using system files. By default, OS are limiting the number of opened files for a user/process. You can see what’s your limit:

``` shell
$ ulimit -n
```

Now increase this limit appending the following:

```
# /etc/security/limits.conf
# Increase the number of simultaneous clients
* soft nofile 100000
* hard nofile 100000
```

> NOTE: It would be nice to use `redis` user instead of `*`, but I couldn't make it work otherwise.

Uncomment the following line in these files:

```
# /etc/pam.d/su 
# /etc/pam.d/sudo
session    required   pam_limits.so
```

The `limits.conf` modified file will now be loaded every time your server boots.
