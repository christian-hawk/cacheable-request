# cacheable-request

> Wrap native HTTP requests with RFC compliant cache support

[![Build Status](https://travis-ci.org/lukechilds/cacheable-request.svg?branch=master)](https://travis-ci.org/lukechilds/cacheable-request)
[![Coverage Status](https://coveralls.io/repos/github/lukechilds/cacheable-request/badge.svg?branch=master)](https://coveralls.io/github/lukechilds/cacheable-request?branch=master)
[![npm](https://img.shields.io/npm/v/cacheable-request.svg)](https://www.npmjs.com/package/cacheable-request)

[RFC 7234](http://httpwg.org/specs/rfc7234.html) compliant HTTP caching for native Node.js HTTP/HTTPS requests. Caching works out of the box with `new Map()` or is easily pluggable with a wide range of cache adapters.

## Install

```shell
npm install --save cacheable-request
```

## Usage

```js
const http = require('http');
const cacheableRequest = require('cacheable-request');

// Then instead of
const opts = {
  host: 'example.com'
};
const req = http.request(opts, cb);
req.end();

// You can do
const cache = new Map();
const opts = {
  host: 'example.com',
  cache: cache
};
const cacheReq = cacheableRequest(http.request, opts, cb);
cacheReq.on('request', req => req.end());

// Or pass in any other http.request API compatible method:
cacheableRequest(https.request, opts, cb);
cacheableRequest(electron.net, opts, cb);
```

Cacheable responses will be stored using the provided cache adapter and returned directly from the cache on future requests if they are still valid. You can check if the response came from a network request or the cache by checking the `fromCache` property on the response.

```js
const cache = new Map();
const opts = {
  host: 'example.com',
  cache: cache
};

cacheableRequest(http.request, opts, response => {
  console.log(response.fromCache);
  // false
}).on('request', req => req.end());

// Then at some point in the future
cacheableRequest(http.request, opts, response => {
  console.log(response.fromCache);
  // true
}).on('request', req => req.end());
```

## Cache Adapters

> TODO

## License

MIT © Luke Childs
