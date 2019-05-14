# mem [![Build Status](https://travis-ci.org/sindresorhus/mem.svg?branch=master)](https://travis-ci.org/sindresorhus/mem)

> [Memoize](https://en.wikipedia.org/wiki/Memoization) functions - An optimization used to speed up consecutive function calls by caching the result of calls with identical input


## Install

```
$ npm install mem
```


## Usage

```js
const mem = require('mem');

let i = 0;
const counter = () => ++i;
const memoized = mem(counter);

memoized('foo');
//=> 1

// Cached as it's the same arguments
memoized('foo');
//=> 1

// Not cached anymore as the arguments changed
memoized('bar');
//=> 2

memoized('bar');
//=> 2
```

##### Works fine with promise returning functions

```js
const mem = require('mem');

let i = 0;
const counter = async () => ++i;
const memoized = mem(counter);

(async () => {
	console.log(await memoized());
	//=> 1

	// The return value didn't increase as it's cached
	console.log(await memoized());
	//=> 1
})();
```


## API

### mem(fn, [options])

#### fn

Type: `Function`

Function to be memoized.

#### options

Type: `Object`

##### cacheKey

Type: `Function`

Determines the cache key for storing the result based on the function arguments. By default, if there's only one argument and it's a [primitive](https://developer.mozilla.org/en-US/docs/Glossary/Primitive), it's used directly as a key, otherwise it's all the function arguments JSON stringified as an array.

You could for example change it to only cache on the first argument `x => JSON.stringify(x)`.

##### cache

Type: `Object`<br>
Default: `new Map()`

Use a different cache storage. Must implement the following methods: `.has(key)`, `.get(key)`, `.set(key, value)`, `.delete(key)`, and optionally `.clear()`. For example uou could use:

- [`quick-lru`](https://github.com/sindresorhus/quick-lru) for a LRU cache;
- [`expiry-map`](https://github.com/SamVerschueren/expiry-map) to set an expiration date;
- `WeakMap` for automatic garbage collection (you'll also need to set `cacheKey: x => x` for this to work).

##### cachePromiseRejection

Type: `boolean`<br>
Default: `false`

Cache rejected promises.

### mem.clear(fn)

Clear all cached data of a memoized function.

#### fn

Type: `Function`

Memoized function.


## Tips

### Cache statistics

If you want to know how many times your cache had a hit or a miss, you can make use of [stats-map](https://github.com/SamVerschueren/stats-map) as a replacement for the default cache.

#### Example

```js
const mem = require('mem');
const StatsMap = require('stats-map');
const got = require('got');

const cache = new StatsMap();
const memGot = mem(got, {cache});

(async () => {
	await memGot('sindresorhus.com');
	await memGot('sindresorhus.com');
	await memGot('sindresorhus.com');

	console.log(cache.stats);
	//=> {hits: 2, misses: 1}
})();
```


## Related

- [p-memoize](https://github.com/sindresorhus/p-memoize) - Memoize promise-returning & async functions


## License

MIT © [Sindre Sorhus](https://sindresorhus.com)
