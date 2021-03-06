**qcloud-connect-redis** is a Redis session store backed by [node_redis](http://github.com/mranney/node_redis), and is insanely fast :). Requires redis >= `2.0.0` for the *SETEX* command.

Update
-----

use [hashring](https://github.com/3rd-Eden/node-hashring) to support multiple redis instance.

Setup
-----

```sh
npm install qcloud-connect-redis express-session
```

Pass the `express-session` store into `qcloud-connect-redis` to create a `RedisStore` constructor.

```js
var session = require('express-session');
var RedisStore = require('qcloud-connect-redis')(session);

app.use(session({
    store: new RedisStore(options),
    secret: 'keyboard cat'
}));
```

Options
-------

A Redis client is required. An existing client can be passed directly using the `client` param or created for you using the `host`, `port`, or `socket` params.

- `client` An existing client
- `host` Redis server hostname
- `port` Redis server portno
- `socket` Redis server unix_socket
- `url` Redis server url

The following additional params may be included:

- `servers` Support multiple redis instance.
-	`ttl` Redis session TTL (expiration) in seconds. Defaults to session.maxAge (if set), or one day.
-	`disableTTL` Disables setting TTL, keys will stay in redis until evicted by other means (overides `ttl`\)
-	`db` Database index to use. Defaults to Redis's default (0).
-	`pass` Password for Redis authentication
-	`prefix` Key prefix defaulting to "sess:"
-	`unref` Set `true` to unref the Redis client. **Warning**: this is [an experimental feature](https://github.com/mranney/node_redis#clientunref).
-	`serializer` An object containing `stringify` and `parse` methods compatible with Javascript's `JSON` to override the serializer used
-	`logErrors` Whether or not to log client errors. (default: `false`\)
	-	If `true`, a default logging function (`console.error`) is provided.
	-	If a function, it is called anytime an error occurs (useful for custom logging)
	-	If `false`, no logging occurs.

```js
// servers example
[
  {
    host: 'string',
    port: 'number',
    pass: 'string'
  },
  {
    host: 'string',
    port: 'number',
    pass: 'string'
  },
  ...
]
```


Any options not included in this list will be passed to the redis `createClient()` method directly.

Custom Redis clients
--------------------

Clients other than `node_redis` will work if they support the same interface. Just pass the client instance as the `client` configuration option. Known supported clients include:

-	[ioredis](https://github.com/luin/ioredis) - adds support for Redis Sentinel and Cluster

FAQ
---

#### How do I handle lost connections to Redis?

By default, the `node_redis` client will [auto-reconnect](https://github.com/mranney/node_redis#overloading) when a connection is lost. But requests may come in during that time. In express, one way this scenario can be handled is including a "session check" after setting up a session (checking for the existence of `req.session`\):

```js
app.use(session( /* setup session here */ ))
app.use(function (req, res, next) {
  if (!req.session) {
    return next(new Error('oh no')) // handle error
  }
  next() // otherwise continue
})
```

If you want to retry, here is [another option](https://github.com/expressjs/session/issues/99#issuecomment-63853989).

License
=======

MIT
