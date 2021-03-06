koa-session [![Build Status](https://secure.travis-ci.org/node-modules/koa-session.png)](http://travis-ci.org/node-modules/koa-session) [![Dependency Status](https://gemnasium.com/node-modules/koa-session.png)](https://gemnasium.com/node-modules/koa-session)
=========

session middlewares fro koa, easy use with cumstom stores such as [redis](https://github.com/node-modules/koa-redis), support defer session getter.

this middleware will only set cookie when session is manual set. each time session modified (and only when session modified), it will reset cookie and session.

you can use the rolling sessions, that will reset the cookie and session for every request which touch the session.

[![NPM](https://nodei.co/npm/koa-sess.png?downloads=true)](https://nodei.co/npm/koa-sess/)

## Usage

### Example

```
var koa = require('koa');
var http = require('http');
var session = require('koa-sess');
var RedisStore = require('koa-redis');

var app = koa();
app.keys = ['keys', 'keykeys'];
app.use(session({
  defer: true,
  store: new RedisStore()
}));

app.use(function *() {
  switch (this.request.url) {
  case '/get':
    get(this);
    break;
  case '/remove':
    remove(this);
    break;
  }
});

function get(ctx) {
  var session = yield ctx.session;  // defer generate session
  session.count = session.count || 0;
  session.count++;
  var body = session.count;
}

function remove(ctx) {
  ctx.session = null;
  ctx.body = 0;
}

app.on('error', function (err) {
  console.error(err.stack);
});

var app = module.exports = http.createServer(app.callback());
app.listen(8080);
```

* After add session middlware, you can use `this.session` to set or get the sessions.
* set `this.session = null;` will destroy this session.
* Alter `this.session.cookie` can handle the cookie options of this user. Also you can use cookie options in session store, for example use `cookie.maxAge` as session store's ttl.

### Options

```
 *`defer`: defer get session, only generate session when you use it by `var session = yield this.session;`, default is false.
 *`key` cookie name defaulting to `koa.sid`
 *`store` session store instance
 *`cookie` session cookie settings, defaulting to
    {path: '/', httpOnly: true, maxAge: null, rewrite: true, signed: true}
 * `rolling`: rolling session, always reset the cookie and sessions, default is false
 ```

* Store can be any Object have `set`, `get`, `destroy` like [MemoryStore](https://github.com/node-modules/koa-session/blob/master/lib/store.js).
* cookie defaulting to

```
{
  path: '/',
  httpOnly: true,
  maxAge: null,
  rewrite: true,
  signed: true
}
```

full list of cookie options, see [jed/cookies](https://github.com/jed/cookies#cookiesset-name--value---options--).

## Session Store

You can use any other store to replace the default MemoryStore, just need these public api:

* `get(sid)`: get session object by sid
* `set(sid, sess, ttl)`: set session object for sid, with a ttl (in ms)
* `destroy(sid)`: destory session for sid

all these api need return a Promise, Thunk or generator.

and use these events to report store's status.

* `connect`
* `disconnect`

You can use [koa-redis](https://github.com/node-modules/koa-redis) to store your session data with redis.

## Licences
(The MIT License)

Copyright (c) 2013 - 2014 dead-horse and other contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
