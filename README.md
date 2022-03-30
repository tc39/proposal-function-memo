# Function.prototype.memo for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2022.

## Rationale
[Function memoization][] is a common technique that caches the results of
function calls and returns the cached results when the same inputs occur again.
These are useful for:

* Optimizing expensive function calls.
* Ensuring that callbacks always return the same [singleton object][].
* [Mutually recursive][] [recursive-descent parsing][].

[function memoization]: https://en.wikipedia.org/wiki/Memoization
[singleton object]: https://en.wikipedia.org/wiki/Singleton_pattern
[mutually recursive]: https://en.wikipedia.org/wiki/Mutual_recursion
[recursive-descente parsing]: https://en.wikipedia.org/wiki/Recursive_descent_parser

Memoization is common but annoying to write. This proposal would add a standard
memoization to the core JavaScript language. Its caching system would be based on WeakMaps and tuples.

## Description
The Function.prototype.memo method would create a new function that calls the
original function at most once for each tuple of given arguments. Any
subsequent calls to the new function with identical arguments would return the
result of the first call with those arguments.

```js
function f (x) { console.log(x); return x * 2; }

const fMemo = f.memo();
fMemo(3); // Prints 3 and returns 6.
fMemo(3); // Does not print anything. Returns 6.
fMemo(2); // Prints 2 and returns 4.
fMemo(2); // Does not print anything. Returns 4.
fMemo(3); // Does not print anything. Returns 6.
```

## Unresolved questions

### Issue 2:
Should `memo` be a prototype method, a static function, a function decorator,
or multiple things?

### Issue 3:
How should cache garbage collection work? (Using WeakMaps for the caches would
be ideal…except that WeakMaps do not support primitives as keys.)

Should we just use Maps and make the developer manage the cache memory
themselves? (See also [LRUMap and LFUMap][].)

There is also the [compositeKeys proposal][].

[LRUMap and LFUMap]: https://github.com/js-choi/policy-map-set
[compositeKeys proposal]: (https://github.com/tc39/proposal-richer-keys/tree/master/compositeKey)

### Issue 4:
If we go with a Map cache, how should we structure the cache? For example, we
could use a tree of Maps, or we could use argument-[tuples][] as keys in one
Map.

[tuples]: https://github.com/tc39/proposal-record-tuple

### Issue 5:
Should we add an LRUMap ([least recently used][]) to the language, like
[Python’s lru_cache][Python functools.lru_cache]? If so, should it be added
with this proposal?

[least recently used]: https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)

## Precedents

* [lodash.memoize](https://lodash.com/docs/4.17.15#memoize)
* [Undescore.js memoize](https://underscorejs.org/#memoize)
* [Python functools.lru_cache][]
* [Wikipedia “function memoization” article][function memoization]

[Python functools.lru_cache]: https://docs.python.org/3/library/functools.html#functools.lru_cache
