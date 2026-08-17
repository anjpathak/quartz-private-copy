---
{"publish":true,"created":"2026-08-13T04:21:47.393Z","modified":"2026-08-17T02:08:34.451Z"}
---

# Utility function implementations — full solutions

Each entry follows the same three-step format we've been using: restate requirements, think through the key design questions, then the code with edge-case tests.

---

## 1. Debounce

### Restate the requirements

> "Debounce takes a function and a delay, and returns a new function that only actually invokes the original after `delay` ms have passed since the _last_ call. Every new call resets the timer. I also want to support an `immediate`/leading-edge flag, and preserve `this` and arguments correctly."

### Key questions

1. How do you "reset the timer" on every call? → keep a `timeoutId` in closure, `clearTimeout` it on every new call before setting a new one.
2. How do you preserve `this` and `arguments` when the debounced function is eventually invoked? → use a regular `function` (not arrow) for the returned wrapper, and `fn.apply(this, args)` inside the timeout callback.
3. What does "immediate" mean exactly? → fire on the _leading_ edge (the very first call) instead of the trailing edge, then ignore further calls until the quiet period passes.

### Code

```javascript
function debounce(fn, delay, immediate = false) {
  let timeoutId = null;

  function debounced(...args) {
    const callNow = immediate && !timeoutId;

    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (!immediate) fn.apply(this, args);
    }, delay);

    if (callNow) fn.apply(this, args);
  }

  debounced.cancel = () => {
    clearTimeout(timeoutId);
    timeoutId = null;
  };

  return debounced;
}

// --- tests ---
const log = debounce((msg) => console.log('fired:', msg), 200);
log('a'); log('b'); log('c');
// only 'fired: c' logs, ~200ms after the last call

const immediateLog = debounce((msg) => console.log('immediate:', msg), 200, true);
immediateLog('x'); immediateLog('y'); immediateLog('z');
// 'immediate: x' logs right away; y and z are swallowed within the 200ms window

log.cancel(); // cancels any pending trailing call
```

---

## 2. Throttle

### Restate the requirements

> "Throttle takes a function and a time limit, and returns a wrapper that invokes the original at most once per `limit` ms, no matter how many times it's called in that window. I want both leading-edge (fire immediately on first call) and trailing-edge (fire once more at the end of the window if calls happened during it) behavior."

### Key questions

1. How is this different from debounce? → debounce waits for silence; throttle guarantees a steady invocation rate even under continuous calls.
2. How do you track "am I in a cooldown window"? → a boolean flag (or a `lastRan` timestamp) plus a `setTimeout` to clear it.
3. How do you support trailing-edge too? → remember if a call came in _during_ the cooldown, and if so, fire once more right when the cooldown ends.

### Code

```javascript
function throttle(fn, limit) {
  let inThrottle = false;
  let lastArgs = null;
  let lastThis = null;

  function throttled(...args) {
    lastArgs = args;
    lastThis = this;

    if (!inThrottle) {
      fn.apply(lastThis, lastArgs); // leading edge
      inThrottle = true;

      setTimeout(() => {
        inThrottle = false;
        if (lastArgs) {
          fn.apply(lastThis, lastArgs); // trailing edge, if calls happened during cooldown
          lastArgs = null;
        }
      }, limit);
    }
  }

  return throttled;
}

// --- tests ---
const onScroll = throttle(() => console.log('scroll handled at', Date.now()), 1000);
// call onScroll() rapidly (e.g. in a loop or fast interval) -> fires immediately once,
// then at most once more per 1000ms window even if called continuously
```

---

## 3. Debounced async search / cancel stale requests

### Restate the requirements

> "I need a search handler that debounces user input, so we don't fire a request on every keystroke, and also cancels/ignores stale in-flight requests — so if an earlier slow request resolves _after_ a newer one, its result doesn't overwrite the newer, correct result on screen."

### Key questions

1. Debouncing alone isn't enough — why? → even with debouncing, a request can still be slow and race with the next debounced request. Network timing isn't guaranteed to match call order.
2. How do you know a response is "stale"? → tag each request with an incrementing ID (or use `AbortController`), and when a response comes back, check if it's still the _latest_ request before using it.
3. What's the simplest correct approach without `AbortController`? → a monotonically increasing request counter compared against the latest counter value at resolve time.

### Code

```javascript
function createDebouncedSearch(searchFn, delay = 300) {
  let timeoutId = null;
  let latestRequestId = 0;

  return function search(query, onResult) {
    clearTimeout(timeoutId);

    timeoutId = setTimeout(async () => {
      const requestId = ++latestRequestId; // this request's unique stamp

      try {
        const result = await searchFn(query);

        if (requestId === latestRequestId) {
          // still the most recent request -- safe to use
          onResult(null, result);
        }
        // else: a newer request has since been fired; silently drop this stale result
      } catch (err) {
        if (requestId === latestRequestId) {
          onResult(err, null);
        }
      }
    }, delay);
  };
}

// --- tests ---
const fakeApi = (q) => new Promise(resolve => {
  const delay = q === 'ab' ? 500 : 100; // simulate 'ab' being slower than 'abc'
  setTimeout(() => resolve(`results for "${q}"`), delay);
});

const search = createDebouncedSearch(fakeApi, 0); // delay=0 to isolate the race-condition test
search('ab', (err, res) => console.log('ab ->', res));   // slow (500ms), would resolve last
search('abc', (err, res) => console.log('abc ->', res)); // fast (100ms), should be the one that renders
// only 'abc -> results for "abc"' should log; 'ab' response arrives later but is discarded as stale
```

**AbortController variant** (better for real `fetch` calls, since it also cancels the network request itself, not just the result):

```javascript
function createDebouncedSearchWithAbort(searchFn, delay = 300) {
  let timeoutId = null;
  let controller = null;

  return function search(query, onResult) {
    clearTimeout(timeoutId);
    if (controller) controller.abort(); // cancel the previous in-flight request

    timeoutId = setTimeout(async () => {
      controller = new AbortController();
      try {
        const result = await searchFn(query, controller.signal);
        onResult(null, result);
      } catch (err) {
        if (err.name !== 'AbortError') onResult(err, null); // ignore expected abort errors
      }
    }, delay);
  };
}
```

---

## 4. Deep flatten (array)

### Restate the requirements

> "Given an arbitrarily nested array, return one flat array with all elements in order. I also want a version that respects a `depth` limit, like native `Array.prototype.flat(depth)`."

### Key questions

1. How do you handle unknown nesting depth? → recursion is the natural fit: if an element is itself an array, recurse into it.
2. What should happen with holes or empty nested arrays? → `[1, [], [2, []]]` should just skip the empty arrays and give `[1, 2]`.
3. How do you add the `depth` parameter version? → pass a decrementing depth counter; stop recursing (just keep the sub-array as-is) once depth hits 0.

### Code

```javascript
function flattenDeep(arr) {
  return arr.reduce((acc, item) => {
    if (Array.isArray(item)) {
      acc.push(...flattenDeep(item)); // recurse fully
    } else {
      acc.push(item);
    }
    return acc;
  }, []);
}

function flatten(arr, depth = 1) {
  if (depth <= 0) return arr.slice(); // no flattening left to do

  return arr.reduce((acc, item) => {
    if (Array.isArray(item)) {
      acc.push(...flatten(item, depth - 1));
    } else {
      acc.push(item);
    }
    return acc;
  }, []);
}

// --- tests ---
console.log(flattenDeep([1, [2, [3, [4, [5]]]]]));  // [1, 2, 3, 4, 5]
console.log(flattenDeep([1, [], [2, []]]));          // [1, 2]
console.log(flattenDeep([1, 'a', [true, null]]));    // [1, 'a', true, null]
console.log(flattenDeep([]));                        // []

console.log(flatten([1, [2, [3, [4]]]], 1));  // [1, 2, [3, [4]]]  -- only one level deep
console.log(flatten([1, [2, [3, [4]]]], 2));  // [1, 2, 3, [4]]
console.log(flatten([1, [2, [3, [4]]]], Infinity)); // [1, 2, 3, 4] -- same as flattenDeep
```

---

## 5. Flatten a deeply nested object

### Restate the requirements

> "Given a nested object like `{a: {b: {c: 1}}, d: 2}`, produce a flat object with dot-notation keys: `{'a.b.c': 1, 'd': 2}`. I need to decide how to handle arrays and null values along the way, and state that decision clearly."

### Key questions

1. How do you tell "should I recurse into this value" apart from "this is a leaf value"? → check `typeof value === 'object' && value !== null && !Array.isArray(value)` — treating arrays as leaf values by default (a reasonable, statable assumption) rather than trying to flatten them with numeric-index keys.
2. What about `null`? → `typeof null === 'object'`, so you must explicitly exclude `null` from "recurse into this," or you'll try to iterate its (nonexistent) keys and get nothing useful, or worse, silently drop the key.
3. What key format do you build as you recurse? → pass an accumulating `prefix` string down through the recursion, appending `.key` each level.

### Code

```javascript
function flattenObject(obj, prefix = '') {
  const result = {};

  for (const key in obj) {
    if (!Object.prototype.hasOwnProperty.call(obj, key)) continue;

    const value = obj[key];
    const newKey = prefix ? `${prefix}.${key}` : key;

    const isPlainObject =
      typeof value === 'object' && value !== null && !Array.isArray(value);

    if (isPlainObject) {
      Object.assign(result, flattenObject(value, newKey)); // recurse, merge results
    } else {
      result[newKey] = value; // leaf value: primitives, arrays, null, functions
    }
  }

  return result;
}

// --- tests ---
console.log(flattenObject({ a: { b: { c: 1 } }, d: 2 }));
// { 'a.b.c': 1, 'd': 2 }

console.log(flattenObject({ a: { b: 1 }, c: [1, 2, 3] }));
// { 'a.b': 1, 'c': [1, 2, 3] }  -- arrays treated as leaf values, per our stated assumption

console.log(flattenObject({ a: null, b: { c: null } }));
// { 'a': null, 'b.c': null }  -- null is a leaf value, not recursed into

console.log(flattenObject({ a: {} }));
// {}  -- empty nested object contributes no keys at all (worth stating this choice out loud)
```

---

## 6. Deep equality check

### Restate the requirements

> "Given two values, determine whether they're structurally equal — same shape and same values at every level — not just reference-equal. Needs to handle nested objects and arrays, treat `NaN === NaN` as true (unlike `===`), and not infinite-loop on circular references."

### Key questions

1. What's the base case? → if `Object.is(a, b)` is true (handles `NaN` correctly, unlike `===`), they're equal — done, no need to recurse.
2. What if types don't match, or one is `null`/not an object? → return `false` immediately; only recurse when both are non-null objects.
3. How do you avoid infinite loops on circular structures? → track pairs of objects you've already compared (e.g. with a `WeakMap` or a `seen` Set), and short-circuit if you hit the same pair again.

### Code

```javascript
function deepEqual(a, b, seen = new WeakMap()) {
  if (Object.is(a, b)) return true; // handles primitives + NaN correctly

  if (typeof a !== 'object' || typeof b !== 'object' || a === null || b === null) {
    return false; // different primitive values, or one is null/not an object
  }

  // array vs plain object shouldn't be considered equal even with matching "contents"
  if (Array.isArray(a) !== Array.isArray(b)) return false;

  // circular reference guard: if we've already started comparing this exact pair, assume equal
  if (seen.has(a) && seen.get(a) === b) return true;
  seen.set(a, b);

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) return false;

  return keysA.every((key) =>
    Object.prototype.hasOwnProperty.call(b, key) &&
    deepEqual(a[key], b[key], seen)
  );
}

// --- tests ---
console.log(deepEqual({ a: 1, b: { c: 2 } }, { a: 1, b: { c: 2 } })); // true
console.log(deepEqual({ a: 1, b: 2 }, { b: 2, a: 1 }));               // true, key order irrelevant
console.log(deepEqual(NaN, NaN));                                    // true
console.log(deepEqual([1, 2], { 0: 1, 1: 2 }));                      // false, array vs object
console.log(deepEqual({ a: 1 }, { a: 1, b: 2 }));                    // false, different key counts

const circA = {}; circA.self = circA;
const circB = {}; circB.self = circB;
console.log(deepEqual(circA, circB)); // true, doesn't infinite loop
```

---

## 7. Curry function

### Restate the requirements

> "Given a function `fn`, return a curried version: it can be called as `fn(a)(b)(c)`, or `fn(a, b, c)`, or any mix like `fn(a, b)(c)` — as soon as enough arguments have been collected to match `fn`'s arity, it actually invokes `fn`."

### Key questions

1. How do you know when "enough" arguments have been collected? → `fn.length` gives you the number of _named_ parameters `fn` declares — compare accumulated args against that.
2. How do you accumulate arguments across multiple calls? → return a new function each time that closes over the args collected _so far_, concatenating new args as they come in.
3. What's the recursive structure? → a helper that takes `accumulatedArgs`; if `accumulatedArgs.length >= fn.length`, call `fn(...accumulatedArgs)`; otherwise return another function that, when called, recurses with the combined args.

### Code

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function (...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

// --- tests ---
function sum3(a, b, c) {
  return a + b + c;
}
const curriedSum = curry(sum3);

console.log(curriedSum(1)(2)(3));    // 6
console.log(curriedSum(1, 2)(3));    // 6
console.log(curriedSum(1, 2, 3));    // 6
console.log(curriedSum(1)(2, 3));    // 6

function noArgs() { return 'called'; }
console.log(curry(noArgs)());        // 'called' -- fn.length is 0, fires immediately
```

---

## 8. Memoize

### Restate the requirements

> "Given a function, return a memoized version that caches results by argument(s), so repeated calls with the same arguments skip recomputation. I want to support a custom key-resolver function, since `JSON.stringify` on the arguments doesn't always give a reliable or unique key."

### Key questions

1. What's the default cache key strategy, and what are its limits? → `JSON.stringify(args)` works for simple primitive args, but breaks for functions, symbols, or when argument order in objects varies unpredictably.
2. Why support a custom `resolver`? → callers know their own argument shapes best — e.g. keying only on an `id` field of an object argument, ignoring the rest.
3. Does memoization work correctly for recursive functions (like fibonacci)? → only if the cache is shared across recursive calls — meaning the function must call the _memoized_ version of itself recursively, not the original.

### Code

```javascript
function memoize(fn, resolver) {
  const cache = new Map();

  return function (...args) {
    const key = resolver ? resolver(...args) : JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// --- tests ---
let callCount = 0;
const slowSquare = memoize((x) => {
  callCount++;
  return x * x;
});

console.log(slowSquare(4), slowSquare(4), slowSquare(5));
console.log('fn actually called', callCount, 'times'); // 2, not 3 -- second call(4) was cached

// recursive memoized fibonacci -- must reference the memoized wrapper inside itself
const memoFib = memoize(function fib(n) {
  if (n <= 1) return n;
  return memoFib(n - 1) + memoFib(n - 2); // recurse via memoFib, so shared cache is used
});
console.log(memoFib(30)); // fast, thanks to shared cache across recursive calls

// custom resolver example: cache only by user id, ignoring other fields
const getUserDisplay = memoize(
  (user) => `${user.name} (${user.id})`,
  (user) => user.id
);
console.log(getUserDisplay({ id: 1, name: 'A' }));
console.log(getUserDisplay({ id: 1, name: 'Changed but same id' })); // cache hit, returns stale 'A' result -- worth discussing this tradeoff live
```

---

## 9. LRU Cache

### Restate the requirements

> "Build an `LRUCache(capacity)` with `get(key)` and `put(key, value)`. When capacity is exceeded, evict the _least recently used_ item. Both operations should be O(1)."

### Key questions

1. What data structure gives O(1) access _and_ preserves usage-order information cheaply? → JavaScript's `Map` preserves **insertion order** during iteration, and importantly, you can `delete` + re-`set` a key to move it to the "most recently used" end — that's the trick.
2. On `get`, what needs to happen even if you're just reading, not writing? → the accessed key must be marked as most-recently-used too (delete + re-insert), since reading counts as "using" it.
3. On `put` when at capacity, how do you find "least recently used" in O(1)? → it's simply the _first_ key in the Map's iteration order (`map.keys().next().value`), since we always move used keys to the end.

### Code

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map(); // insertion order = recency order (oldest first)
  }

  get(key) {
    if (!this.cache.has(key)) return -1;

    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value); // move to the "most recently used" end
    return value;
  }

  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key); // remove old position so re-insertion moves it to the end
    } else if (this.cache.size >= this.capacity) {
      const lruKey = this.cache.keys().next().value; // first key = least recently used
      this.cache.delete(lruKey);
    }
    this.cache.set(key, value);
  }
}

// --- tests ---
const lru = new LRUCache(2);
lru.put(1, 'a');
lru.put(2, 'b');
console.log(lru.get(1));   // 'a' -- also marks key 1 as most recently used
lru.put(3, 'c');           // capacity exceeded; key 2 is now LRU (since 1 was just touched), gets evicted
console.log(lru.get(2));   // -1 -- evicted
console.log(lru.get(1));   // 'a' -- still present
console.log(lru.get(3));   // 'c'

const lru1 = new LRUCache(1);
lru1.put('x', 1);
lru1.put('y', 2); // evicts 'x' immediately, capacity is 1
console.log(lru1.get('x')); // -1
console.log(lru1.get('y')); // 2
```

---

## 10. Custom EventEmitter class

### Restate the requirements

> "Build an EventEmitter with `.on(event, handler)`, `.off(event, handler)`, `.once(event, handler)`, and `.emit(event, ...args)`. Multiple listeners per event should all fire. `.off` should remove only the specific handler passed in. If one listener throws, the others for that event should still run. `.once` handlers should only fire on the next emit, then auto-remove themselves."

### Key questions

1. What's the underlying data structure? → a `Map` (or plain object) from event name → array of listener functions.
2. How do you implement `.once` without a separate code path for "once" vs "on" listeners? → wrap the handler in another function that calls the original _and then_ calls `.off` on itself, then register that wrapper via the normal `.on`.
3. What happens if a listener mutates the listeners array _while_ `.emit` is iterating it (e.g. a listener calls `.off` on itself mid-emit)? → iterate over a **copy** of the listeners array (`[...listeners]`), so mutations to the original array during iteration don't skip or duplicate anyone.
4. How do you isolate errors between listeners? → wrap each individual listener call in its own `try/catch` inside the loop, not one `try/catch` around the whole loop.

### Code

```javascript
class EventEmitter {
  constructor() {
    this.events = new Map(); // eventName -> array of handler functions
  }

  on(event, handler) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event).push(handler);
    return this; // allow chaining: emitter.on(...).on(...)
  }

  off(event, handler) {
    if (!this.events.has(event)) return this;
    const handlers = this.events.get(event).filter((h) => h !== handler);
    this.events.set(event, handlers);
    return this;
  }

  once(event, handler) {
    const wrapper = (...args) => {
      this.off(event, wrapper); // remove self before running, so re-entrant emits don't refire it
      handler.apply(this, args);
    };
    this.on(event, wrapper);
    return this;
  }

  emit(event, ...args) {
    if (!this.events.has(event)) return false;

    // iterate a COPY, so a listener calling .off() mid-emit doesn't corrupt this iteration
    const handlers = [...this.events.get(event)];

    handlers.forEach((handler) => {
      try {
        handler.apply(this, args);
      } catch (err) {
        // isolate: one listener throwing shouldn't stop the others from running
        console.error(`Error in listener for "${event}":`, err);
      }
    });

    return true;
  }
}

// --- tests ---
const emitter = new EventEmitter();

function onData(data) { console.log('handler 1:', data); }
function onDataAgain(data) { console.log('handler 2:', data); }

emitter.on('data', onData);
emitter.on('data', onDataAgain);
emitter.emit('data', 'hello'); // both fire

emitter.off('data', onData);
emitter.emit('data', 'world'); // only handler 2 fires now

emitter.once('greet', (name) => console.log('greeted:', name));
emitter.emit('greet', 'Anjul'); // fires
emitter.emit('greet', 'Anjul'); // does NOT fire again

emitter.on('boom', () => { throw new Error('listener failed'); });
emitter.on('boom', () => console.log('this still runs despite the error above'));
emitter.emit('boom'); // error logged, but second listener still runs

emitter.emit('nonexistent'); // returns false, doesn't throw
```

---

## 11. Pub/Sub module

### Restate the requirements

> "Build a standalone (not class-based) module with `publish(topic, data)` and `subscribe(topic, fn)`. `subscribe` should return an unsubscribe function. Multiple subscribers per topic should all fire. Unsubscribing mid-publish shouldn't break the current publish cycle. Publishing to a topic with no subscribers should be a safe no-op."

### Key questions

1. How is this actually different from the EventEmitter above? → conceptually almost identical under the hood — the real difference is API shape (module-level functions vs a class instance) and, often, that `subscribe` directly returns the unsubscribe function rather than requiring you to call `.off()` separately with the original handler reference.
2. Why is returning an unsubscribe function from `subscribe` nicer? → the caller doesn't need to keep a separate reference to the handler around just to unsubscribe later — the closure handles that for them.
3. Same mid-iteration mutation concern as EventEmitter — same fix (copy the array before iterating).

### Code

```javascript
const PubSub = (function () {
  const topics = new Map(); // topic -> array of subscriber functions

  function subscribe(topic, fn) {
    if (!topics.has(topic)) {
      topics.set(topic, []);
    }
    topics.get(topic).push(fn);

    // return an unsubscribe function closing over exactly this fn + topic
    return function unsubscribe() {
      const subs = topics.get(topic);
      if (!subs) return;
      topics.set(topic, subs.filter((s) => s !== fn));
    };
  }

  function publish(topic, data) {
    const subs = topics.get(topic);
    if (!subs || subs.length === 0) return; // safe no-op

    [...subs].forEach((fn) => {
      try {
        fn(data);
      } catch (err) {
        console.error(`Error in subscriber for "${topic}":`, err);
      }
    });
  }

  return { subscribe, publish };
})();

// --- tests ---
const unsubA = PubSub.subscribe('order.created', (data) => console.log('A heard:', data));
const unsubB = PubSub.subscribe('order.created', (data) => console.log('B heard:', data));

PubSub.publish('order.created', { id: 101 }); // both A and B log

unsubA(); // remove only A's subscription
PubSub.publish('order.created', { id: 102 }); // only B logs now

PubSub.publish('nothing.subscribed', { x: 1 }); // no-op, no error

// mid-publish unsubscribe safety
let unsubSelf;
unsubSelf = PubSub.subscribe('tick', () => {
  console.log('firing once, then unsubscribing self');
  unsubSelf();
});
PubSub.subscribe('tick', () => console.log('second listener still runs this cycle'));
PubSub.publish('tick'); // both fire this time (we copied the array before iterating)
PubSub.publish('tick'); // only 'second listener' fires now
```

---

## Practice order suggestion

Given what's already come up for you (event emitter) and what's most likely to recur:

1. **EventEmitter + Pub/Sub** — you already know one variant is coming; make sure you can write both the class-based and module-based versions cold, and clearly articulate the difference if asked.
2. **Debounce + Throttle** — near-guaranteed in a frontend-heavy round; the debounced-search variant is a strong "applied" follow-up they might layer on top live.
3. **Deep equality + Deep flatten (object)** — these test whether you _reason_ about object traversal correctly, not just memorize a template; expect a live twist (e.g. "now handle `Date` objects" or "what about `Map`/`Set` values?").
4. **Curry + Memoize** — slightly less common as cold-starts, but excellent "why would you use this" discussion topics if you finish early.
5. **LRU Cache** — more of a Round 2/data-structures flavor question; good to have but lower priority than the others for a frontend-fundamentals screen.
