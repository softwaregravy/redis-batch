redis-batch
===========

[![Build Status](https://magnum.travis-ci.com/segmentio/redis-batch.png?token=exthRyNRTLjkmpmb4FUi&branch=master)](https://magnum.travis-ci.com/segmentio/cloud)

Redis Batch wraps a [node_redis](https://github.com/mranney/node_redis) instance, and batches write commands like hincrby, incrby and sadd to Redis. For systems generating *lots* of writes to Redis, this can be a significant performance increase for Redis and for the machines issuing the write commands.

When we deployed redis-batch within Segment's server cluster:
+ **Redis** ops/second dropped by a factor of 4
+ **Servers issuing commands** CPU dropped by 50%

YMMV.

## Commands supported
Redis-batch only supports "write" commands that make sense to be batched. Of the 150+ Redis commands, redis-batch is only intended to support 34. Currently it supports 12. Not all of them are supported yet, feel free to pull request!

#### Incrementing
- [x] decr, decrby
- [x] incr, incrby
- [ ] incrbyfloat
- [x] hincr, hincrby
- [ ] hincrbyfloat
- [ ] zincrby

#### Keys
- [ ] append
- [ ] del
- [x] set, mset
- [ ] setbit
- [ ] rename
- [ ] hdel
- [ ] hset, hmset

#### Lists
- [x] lpush
- [ ] rpush
- [ ] lrem
- [ ] lset
- [ ] ltrim

#### Sets
- [ ] srem
- [x] sadd
- [ ] smove
- [ ] zadd
- [ ] zrem
- [ ] zremrangebyrank
- [ ] zremrangebyscore

#### Persistance
- [x] pexpire, expire
- [ ] pexpireat, expireat
- [x] persist

## Usage

```javascript
var RedisBatch = require('redis-batch');

var redisBatch = new RedisBatch(redis); // redis instance created elsewhere
var redisBatch = new RedisBatch(redis, { flushAfter: 3000 });

/**
 * hincrby batches by key:field
 * This will become a single "hincrby warehouse pants -6" when sent to redis.
 */

redisBatch.hincrby('warehouse', 'pants', -2)
    .hincrby('warehouse', 'pants', -1)
    .hincrby('warehouse', 'pants', -3);

/**
 * incrby batches by key
 * This will become a single "incrby projectcounter 22" when sent to redis.
 */

redisBatch.incrby('projectcounter', 12)
    .incrby('projectcounter', 6)
    .incrby('projectcounter', 4);

/**
 * pexpire de-duplicates by key
 * This will become a single "pexpire key 1000" when sent to redis.
 */

redisBatch.pexpire('key', 1000)
    .pexpire('key', 1000)
    .pexpire('key', 1000);

/**
 * sadd batches by key
 * This will become a single "sadd key member1 member2 member3" when sent to redis.
 */

redisBatch.sadd('key', 'member1')
    .sadd('key', 'member2')
    .sadd('key', 'member3');
```

You can intermingle sadd, incrby and hincrby as you desire.

## License (MIT)

```
WWWWWW||WWWWWW
 W W W||W W W
      ||
    ( OO )__________
     /  |           \
    /o o|    MIT     \
    \___/||_||__||_|| *
         || ||  || ||
        _||_|| _||_||
       (__|__|(__|__|
```

Copyright (c) 2013 Segment.io Inc. <friends@segment.io>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
