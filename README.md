# cacheable-request

> Wrap native HTTP requests with RFC compliant cache support

[![Build Status](https://travis-ci.org/lukechilds/cacheable-request.svg?branch=master)](https://travis-ci.org/lukechilds/cacheable-request)
[![Coverage Status](https://coveralls.io/repos/github/lukechilds/cacheable-request/badge.svg?branch=master)](https://coveralls.io/github/lukechilds/cacheable-request?branch=master)
[![npm](https://img.shields.io/npm/v/cacheable-request.svg)](https://www.npmjs.com/package/cacheable-request)

[RFC 7234](http://httpwg.org/specs/rfc7234.html) compliant HTTP caching for native Node.js HTTP/HTTPS requests. Caching works out of the box in memory or is easily pluggable with a wide range of storage adapters.

## Install

```shell
npm install --save cacheable-request
```

## Usage

```js
const http = require('http');
const CacheableRequest = require('cacheable-request');

// Then instead of
const req = http.request('http://example.com', cb);
req.end();

// You can do
const cacheableRequest = new CacheableRequest(http.request);
const cacheReq = cacheableRequest('http://example.com', cb);
cacheReq.on('request', req => req.end());
// Future requests to 'example.com' will be returned from cache if still valid

// You pass in any other http.request API compatible method to be wrapped with cache support:
const cacheableRequest = new CacheableRequest(https.request);
const cacheableRequest = new CacheableRequest(electron.net);
```

## Storage Adapters

`cacheable-request` uses [Keyv](https://github.com/lukechilds/keyv) to support a wide range of storage adapters.

For example, to use Redis as a cache backend, you just need to install the official Redis Keyv storage adapter:

```
npm install --save @keyv/redis
```

And then you can pass `CacheableRequest` your connection string:

```js
const cacheableRequest = new CacheableRequest(http.request, 'redis://user:pass@localhost:6379');
```

[View all official Keyv storage adapters.](https://github.com/lukechilds/keyv#official-storage-adapters)

Keyv also supports anything that follows the Map API so it's easy to write your own storage adapter or use a third-party solution.

e.g The following are all valid storage adapters

```js
const storageAdapter = new Map();
// or
const storageAdapter = require('./my-storage-adapter');
// or
const QuickLRU = require('quick-lru');
const storageAdapter = new QuickLRU({ maxSize: 1000 });

const cacheableRequest = new CacheableRequest(http.request, storageAdapter);
```

View the [Keyv docs](https://github.com/lukechilds/keyv) for more information on how to use storage adapters.

## API

### new cacheableRequest(request, [storageAdapter])

Returns the provided request function wrapped with cache support.

#### request

Type: `function`

Request function to wrap with cache support. Should be [`http.request`](https://nodejs.org/api/http.html#http_http_request_options_callback) or a similar API compatible request function.

#### storageAdapter

Type: `Keyv storage adapter`<br>
Default: `new Map()`

A [Keyv](https://github.com/lukechilds/keyv) storage adapter instance, or connection string if using with an official Keyv storage adapter.

### Instance

#### cacheableRequest(opts, [cb])

Returns an event emitter.

##### opts

Type: `object`, `string`

Any of the default request functions options plus:

###### opts.cache

Type: `boolean`<br>
Default: `true`

If the cache should be used. Setting this to false will completely bypass the cache for the current request.

###### opts.strictTtl

Type: `boolean`<br>
Default: `false`

If set to `false`, after a cached resource's TTL expires it is kept in the cache and will be revalidated on the next request with `If-None-Match`/`If-Modified-Since` headers.

If set to `true` once a cached resource has expired it is deleted and will have to be re-requested.

##### cb

Type: `function`

The callback function which will receive the response as an argument.

The response can be either a [Node.js HTTP response stream](https://nodejs.org/api/http.html#http_class_http_incomingmessage) or a [responselike object](https://github.com/lukechilds/responselike). The response will also have a `fromCache` property set with a boolean value.

##### .on('request', request)

`request` event to get the request object of the request.

**Note:** This event will only fire if an HTTP request is actually made, not when a response is retrieved from cache. However, you should always handle the `request` event to end the request and handle any potential request errors.

##### .on('response', response)

`response` event to get the response object from the HTTP request or cache.

##### .on('error', error)

`error` event emitted in case of an error with the cache.

**Note:** You should still handle request errors in the `request` event. e.g:

```js
cacheableRequest('example.com', cb)
  .on('error', handleCacheError)
  .on('request', req => {
    req.on('error', handleRequestError);
    req.end();
  });
```

## License

MIT © Luke Childs
