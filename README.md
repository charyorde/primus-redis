# primus-redis
[![Build Status](https://travis-ci.org/mmalecki/primus-redis.png)](https://travis-ci.org/mmalecki/primus-redis)

`primus-redis` is a Redis store for [Primus](https://github.com/primus/primus).
It takes care of distributing messages to other instances using [Redis Pub/Sub](http://redis.io/topics/pubsub).

*Note:* this is a very simple module for broadcasting all your messages over
Redis to *all* connected clients. If you're looking for room functionality,
see [`primus-redis-rooms`](https://github.com/mmalecki/primus-redis-rooms).

## Usage

### Single Redis instance
You can use `primus-redis` with a single Redis instance, but it's not
recommended in production environment, since it makes Redis a single point of
failure.


```js
var http = require('http'),
    Primus = require('primus'),
    PrimusRedis = require('primus-redis');

var server = http.createServer();
var primus = new Primus(server, {
  redis: {
    host: 'localhost',
    port: 6379,
    channel: 'primus' // Optional, defaults to `'primus`'
  },
  transformer: 'websockets'
});
primus.use('Redis', PrimusRedis);

//
// This'll take care of sending the message to all other instances connected
// to the same Redis channel.
//
primus.write('Hello world!'); 
```

### Sentinel
[Redis Sentinel](http://redis.io/topics/sentinel) is a failover mechanism
built into Redis.

When using Sentinel, Redis client will automatically reconnect to new master
server when current one goes down.

```js
var http = require('http'),
    Primus = require('primus'),
    PrimusRedis = require('primus-redis');

var server = http.createServer();
var primus = new Primus(server, {
  redis: {
    sentinel: true,
    endpoints: [
      { host: 'localhost', port: 26379 },
      { host: 'localhost', port: 26380 },
      { host: 'localhost', port: 26381 }
    ],
    masterName: 'mymaster'
    channel: 'primus' // Optional, defaults to `'primus`'
  },
  transformer: 'websockets'
});
primus.use('Redis', PrimusRedis);
