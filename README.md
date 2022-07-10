# Function.prototype.memo for JavaScript
ECMAScript Stage-0 Proposal.

Champions: Hemanth HM; J. S. Choi.

## Rationale
[Function memoization][] is a common technique that caches the results of
function calls and returns the cached results when the same inputs occur again.
These are useful for:

* Optimizing expensive function calls (e.g., factorials, Fibonacci numbers) in a space–time tradeoff.
* Caching state→UI calculations (e.g., in [React’s useMemo][]).
* Ensuring that callbacks always return the same [singleton object][].
* [Mutually recursive][] [recursive-descent parsing][].
* Implementing [hashlife from cellular automata][hashlife].
* [Materializing views for database queries][materialized views].
* [Tabling in logic programming][logic tabling].

[function memoization]: https://en.wikipedia.org/wiki/Memoization
[React’s useMemo]: https://reactjs.org/docs/hooks-reference.html#usememo
[singleton object]: https://en.wikipedia.org/wiki/Singleton_pattern
[mutually recursive]: https://en.wikipedia.org/wiki/Mutual_recursion
[recursive-descent parsing]: https://en.wikipedia.org/wiki/Recursive_descent_parser
[hashlife]: https://en.wikipedia.org/wiki/Hashlife
[materialized views]: https://en.wikipedia.org/wiki/Materialized_view
[logic tabling]: https://www.metalevel.at/prolog/memoization

Memoization is useful, common, but annoying to write.
We propose exploring the addition of a memoization API to the JavaScript language.

If this proposal is approved for Stage 1, then we would explore various
directions for the API’s design. We would also assemble as many real-world use
cases as possible and shape our design to fulfill them.

In addition, if both [proposal-policy-map-set][] and this proposal are approved for
Stage 1, then we would explore how memoized functions could use these data
structures to control their caches’ memory usage.

[proposal-policy-map-set]: https://github.com/js-choi/proposal-policy-map-set

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

Additionally, we may also add a function-decorator version: `@Function.memo`.
This would make it easier to apply memoization to function declarations:

```js
@Function.memo
function f (x) { console.log(x); return x * 2; }
```

Either version would work with recursive functions:

```js
// Version with prototype method:
const getFibonacci = (function (n) {
  if (n < 2) {
    return n;
  } else {
    return getFibonacci(n - 1) +
      getFibonacci(n - 2);
  }
}).memo();
console.log(getFibonacci(100));

// Version with function decorator:
@Function.memo
function getFibonacci (n) {
  if (n < 2) {
    return n;
  } else {
    return getFibonacci(n - 1) +
      getFibonacci(n - 2);
  }
}
console.log(getFibonacci(100));
```

### Result caches
The developer would be able to pass an optional `cache` argument. This argument
must be a Map-like object with `.has`, `.get`, and `.set` methods. In
particular, there is a [proposal for Map-like objects with cache-replacement
policies like LRUMap][proposal-policy-map-set], which would allow developers to
easily specify that the memoized function use a memory-constrained cache.

[proposal-policy-map-set]: https://github.com/js-choi/proposal-policy-map-set

There are at least two possible ways we could design the `cache` parameter; see
[Issue 3][] and [Issue 4][].

#### Tuple keys?
A: We could use [tuples][] as the cache’s keys. Each tuple represents a
function call to the memoized function, and the tuple would be of the form `#[thisVal, newTargetVal, ...args]`.

Object values would be replaced by symbols that uniquely identify that object.
(Tuples cannot directly contain objects. The memoized function’s closure would
close over an internal WeakMap that maps objects to their symbols.)

```js
const cache = new LRUMap(256);
const f = (function f (arg0) { return this.x + arg0; }).memo(cache);
const o0 = { x: 'a' }, o1 = { x: 'b' };
f.call(o0, 0); // Returns 'a0'.
f.call(o1, 1); // Returns 'b1'.
```

Now cache would be `LRUMap(2) { #[s0, undefined, 0] ⇒ 'a0', #[s1, undefined, 1]
⇒ 'b1' }`, where `s0` and `s1` are unique symbols. `f`’s closure would
internally close over a `WeakMap { o0 ⇒ s0, o1 ⇒ s1 }`.

The default behavior of `memo` (i.e., without giving a `cache` argument) is uncertain (see [Issue 3][]). It probably would be simply be an unbounded ordinary Map. (WeakMaps cannot contain tuples as their keys.)

[tuples]: https://github.com/tc39/proposal-record-tuple

#### Composite keys?
B: Another choice for cache’s keys is [composite keys][]. Each composite key
represents a function call to the memoized function, and the composite key would be of the form `compositeKey(thisVal, newTargetVal, ...args)`.

```js
const cache = new LRUMap(256);
const f = (function f (arg0) { return this.x + arg0; }).memo(cache);
const o0 = { x: 'a' }, o1 = { x: 'b' };
f.call(o0, 0); // Returns 'a0'.
f.call(o1, 1); // Returns 'b1'.
```

Now cache would be `LRUMap(2) { compositeKey(o0, undefined, 0) ⇒ 'a0',
compositeKey(o1, undefined, 1) ⇒ 'b1' }`.

The default behavior of `memo` (i.e., without giving a `cache` argument) is
uncertain (see [Issue 3][]). It probably would simply be a WeakMap,
which would be able to contain composite keys as their keys.

[composite keys]: https://github.com/tc39/proposal-richer-keys/tree/master/compositeKey

## Unresolved questions

### [Issue 2][]:
Should `memo` be a prototype method, a static function, a function decorator,
or multiple things?

### [Issue 3][]:
How should cache garbage collection work? (Using WeakMaps for the caches would
be ideal…except that WeakMaps do not support primitives as keys.)

Should we just use Maps and make the developer manage the cache memory
themselves? (See also [LRUMap and LFUMap][].)

There is also the [compositeKeys proposal][].

[LRUMap and LFUMap]: https://github.com/js-choi/proposal-policy-map-set
[compositeKeys proposal]: (https://github.com/tc39/proposal-richer-keys/tree/master/compositeKey)

### [Issue 4][]:
If we go with a Map cache, how should we structure the cache? For example, we
could use a tree of Maps, or we could use argument-[tuples][] as keys in one
Map.

[tuples]: https://github.com/tc39/proposal-record-tuple

### [Issue 5][]:
How should function calls be considered “equivalent”? How are values compared
(e.g., with SameValue like `===` or SameValueZero like `Map.get`)? Are the
`this`-binding receiver and the `new.target` value also used in comparison?

## Precedents

* [lodash.memoize](https://lodash.com/docs/4.17.15#memoize)
* [Undescore.js memoize](https://underscorejs.org/#memoize)
* [Python functools.lru_cache][]
* [Wikipedia “function memoization” article][function memoization]

[Python functools.lru_cache]: https://docs.python.org/3/library/functools.html#functools.lru_cache

[Issue 2]: https://github.com/js-choi/proposal-function-memo/issues/2
[Issue 3]: https://github.com/js-choi/proposal-function-memo/issues/3
[Issue 4]: https://github.com/js-choi/proposal-function-memo/issues/4
[Issue 5]: https://github.com/js-choi/proposal-function-memo/issues/5
