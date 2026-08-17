---
{"publish":true,"created":"2026-08-12T10:18:20.697Z","modified":"2026-08-17T02:08:29.993Z"}
---

# JavaScript hands-on practice — Round 1/2 prep

Each question has: what to build, requirements, and edge cases to test yourself against. Write these from scratch in a plain editor (no library helpers) — that's what they'll expect live.

---

## A. Polyfills

### A1. `Array.prototype.map`

Implement `Array.prototype.myMap(callback, thisArg)`.

- Requirements: callback receives `(element, index, array)`; supports `thisArg`; returns a new array; must skip holes in sparse arrays (like real `map`).
- Edge cases:
  - `[1,2,3].myMap(x => x * 2)` → `[2,4,6]`
  - `[].myMap(x => x)` → `[]`
  - `[1,,3].myMap(x => x * 2)` → holes preserved, not `NaN`
  - Callback that mutates the original array mid-iteration — decide/explain behavior
  - Non-array-like `this` (e.g. called on a string) — should still work via array-like duck typing

### A2. `Array.prototype.filter`

- Requirements: callback `(element, index, array)`, returns new array, skips holes.
- Edge cases: empty array, all elements filtered out, callback using `index`, sparse array.

### A3. `Array.prototype.reduce`

- Requirements: support optional `initialValue`; if omitted, use first element as initial accumulator and start iterating from index 1; throw `TypeError` on empty array with no initial value.
- Edge cases:
  - `[1,2,3].myReduce((a,b) => a+b)` → `6`
  - `[].myReduce((a,b) => a+b)` → throws TypeError
  - `[].myReduce((a,b) => a+b, 10)` → `10`
  - `[1,2,3].myReduce((a,b) => a+b, 10)` → `16`

### A4. `Function.prototype.bind`

Implement `Function.prototype.myBind(context, ...args)`.

- Requirements: returns a new function with `this` bound; supports partial application (preset args + call-time args); must work correctly when the bound function is later called with `new` (the `new` binding should override the bound `this`, and `instanceof` should still work).
- Edge cases:
  - `const bound = fn.myBind(obj, 1, 2); bound(3)` → calls fn with (1,2,3) and this=obj
  - `new BoundFn(...)` should still construct properly, ignoring the bound `this`
  - Binding a bound function again

### A5. `Function.prototype.call` and `.apply`

Implement both from scratch (classic interview ask — usually via a unique Symbol property trick).

- Requirements: `myCall(context, ...args)`, `myApply(context, argsArray)`; handle `context` being `null`/`undefined` (should default to global object in non-strict mode).
- Edge cases: calling on a context that already has a property with your temp key name (avoid collisions with `Symbol()`), passing no args.

### A6. `Object.create`

Implement without using the built-in.

- Requirements: returns new object with given prototype; support optional second argument for property descriptors (stretch goal).
- Edge cases: `Object.create(null)` (no prototype at all) — most candidates forget this case.

### A7. `Promise.all`, `Promise.race`, `Promise.allSettled` (polyfills)

- Requirements for `Promise.all`: resolves with array of results in input order (not completion order); rejects immediately on first rejection; must handle non-promise values in the array (e.g. plain numbers) by treating them as already resolved.
- Requirements for `Promise.race`: settles (resolve or reject) as soon as the first promise settles.
- Requirements for `Promise.allSettled`: never rejects; resolves with array of `{status, value}` or `{status, reason}`.
- Edge cases:
  - Empty array input for all three (what should each resolve/return with?)
  - Mixed array of promises and plain values
  - One promise rejects immediately, others are slow — verify `all` rejects fast without waiting
  - Order preservation when promises resolve out of order

### A8. Deep clone (`structuredClone` polyfill, simplified)

- Requirements: handles nested objects/arrays, primitive values, `Date`, circular references (don't infinite-loop!).
- Edge cases:
  - `{a: {b: {c: 1}}}` — nested mutation isolation (mutating clone shouldn't affect original)
  - Circular reference: `const o = {}; o.self = o;` — must not stack-overflow
  - Arrays with objects inside
  - `null`, `undefined`, functions (decide: skip, or throw, or copy reference — state your assumption)
  - `Date` objects — clone should be a new Date instance with same value, not same reference

---

## B. Async patterns

### B1. `getAllLinks` / async DOM/data traversal with concurrency control

Build a function that fetches a list of URLs but only allows `N` concurrent requests at a time (a concurrency-limited batch fetcher / async pool).

- Requirements: `asyncPool(poolLimit, items, iteratorFn)` — processes `items` through `iteratorFn` (which returns a promise), never running more than `poolLimit` at once; returns results in original input order.
- Edge cases:
  - `poolLimit` greater than array length
  - `poolLimit` of 1 (fully sequential)
  - One item's promise rejects — decide: fail fast, or collect errors and continue (state your choice, and consider implementing both)
  - Empty items array

### B2. Retry with exponential backoff

- Requirements: `retry(fn, retries, delayMs)` retries an async function on failure, with exponentially increasing delay (and optional jitter) between attempts; gives up after `retries` attempts and throws the last error.
- Edge cases:
  - Function succeeds on the first try (no delay should happen)
  - Function always fails — verify total attempt count matches `retries`
  - `retries = 0`

### B3. Debounced async search / cancel stale requests

- Requirements: build a search-as-you-type handler that debounces input, cancels/ignores stale in-flight requests (using `AbortController` or a request-id guard), and only renders the result of the _latest_ query even if an earlier slow request resolves last.
- Edge cases:
  - User types fast, then stops — only the last request's result should render
  - An earlier (stale) request resolves _after_ a later one — must not overwrite the newer result
  - User clears the input mid-request

### B4. Promise-based `sleep` and a simple task queue

- Requirements: `sleep(ms)` returns a promise that resolves after `ms`. Then build a `TaskQueue` class that runs async tasks one at a time in submission order, exposing `.add(taskFn)` and returning a promise per task that resolves with that task's result.
- Edge cases: tasks added while the queue is already processing; a task that throws shouldn't halt the rest of the queue.

### B5. Custom `Promise` implementation (from scratch, no native `Promise`)

A common "prove you understand promises" ask — implement `then`, `catch`, `finally`, resolve/reject state transitions, and chaining.

- Requirements: states `pending → fulfilled/rejected` (one-way, one-time); `.then()` returns a new promise (chainable); handles async resolution (resolving inside a `setTimeout`); handles a `.then()` handler that itself returns a promise (thenable chaining/flattening).
- Edge cases:
  - Calling `resolve()` twice — second call should be a no-op
  - `.then()` called after the promise has already settled — handler should still fire (async, e.g. via microtask)
  - A `.then` handler throwing an error — should propagate to `.catch`

---

## C. Utility functions

### C1. Debounce

- Requirements: `debounce(fn, delay)` — only invoke `fn` after `delay` ms have passed since the _last_ call; support optional `immediate`/leading-edge flag; preserve `this` and arguments.
- Edge cases:
  - Rapid-fire calls — only the last one should execute
  - `immediate: true` — first call fires immediately, then subsequent calls within the window are ignored until the window passes
  - Cancel method (`debounced.cancel()`) — bonus, often asked as a follow-up

### C2. Throttle

- Requirements: `throttle(fn, limit)` — invoke `fn` at most once per `limit` ms, regardless of how many times it's called; support both leading and trailing-edge invocation (bonus).
- Edge cases:
  - Calls spaced closer than `limit` — only first (or first+last depending on config) fires
  - Verify the difference from debounce with a rapid burst followed by a pause

### C3. Deep flatten (array)

- Requirements: `flattenDeep(arr)` flattens arbitrarily nested arrays into one flat array; also implement a version with a `depth` parameter (like native `Array.prototype.flat(depth)`).
- Edge cases:
  - `[1, [2, [3, [4, [5]]]]]` → `[1,2,3,4,5]`
  - Empty nested arrays: `[1, [], [2, []]]` → `[1,2]`
  - Mixed types: `[1, 'a', [true, null]]`

### C4. Flatten a deeply nested object

- Requirements: `flattenObject(obj)` converts `{a: {b: {c: 1}}, d: 2}` into `{'a.b.c': 1, 'd': 2}`.
- Edge cases:
  - Arrays as values — decide: treat as leaf value, or flatten with index keys (`a.0`, `a.1`) — state your assumption
  - `null` values (don't treat as an object to recurse into)
  - Empty nested object: `{a: {}}` — decide how to represent (drop key, or `a: {}`)

### C5. Deep equality check

- Requirements: `deepEqual(a, b)` compares nested objects/arrays for structural equality (not reference equality).
- Edge cases:
  - `{a:1, b:{c:2}}` vs same shape different reference → true
  - Different key order in objects → still true
  - `NaN` vs `NaN` → should be true (unlike `===`)
  - Arrays vs objects with same-looking content → false
  - Circular references — should not infinite loop

### C6. Curry function

- Requirements: `curry(fn)` transforms a function so it can be called as `fn(a)(b)(c)` or `fn(a,b,c)` or `fn(a,b)(c)` — any combination — based on `fn.length` (arity).
- Edge cases: function with 0 args, calling with more args than arity, currying a function that itself takes a variable number of arguments (harder — usually out of scope, but be ready to discuss).

### C7. Memoize

- Requirements: `memoize(fn)` caches results based on arguments; support a custom key-resolver function as a second argument (since default `JSON.stringify(args)` doesn't handle all cases like functions/symbols as args).
- Edge cases: function with multiple arguments, object arguments (reference vs value equality — discuss trade-off), recursive functions (e.g. memoized fibonacci) — cache must be shared across recursive calls.

### C8. LRU Cache

- Requirements: `LRUCache(capacity)` with `.get(key)` and `.put(key, value)`, evicting the least-recently-used item when capacity is exceeded, in O(1) time (typically Map + manual ordering, since JS `Map` preserves insertion order).
- Edge cases:
  - `get` on a missing key
  - `put` on an existing key should update value and refresh recency
  - Capacity of 1
  - Eviction order after a mix of gets and puts

### C9. Custom `EventEmitter` class

- Requirements: `.on(event, handler)`, `.off(event, handler)`, `.once(event, handler)`, `.emit(event, ...args)`; support multiple listeners per event; `.off` should remove only the specific handler passed in; an error thrown in one listener shouldn't stop other listeners for the same event from running.
- Edge cases:
  - Emitting an event with no listeners registered (shouldn't throw)
  - Calling `.off` with a handler that was never registered
  - A listener that calls `.off` on itself _while_ it's being invoked (mutating the listener array mid-iteration)
  - `.once` handler must not fire on subsequent emits
  - Same handler function registered twice via `.on` — should it fire twice? (decide and be ready to defend it)

### C10. Simple Pub/Sub module (as an alternative framing of C9)

- Requirements: standalone `publish(topic, data)` / `subscribe(topic, fn)` functions (module-level, not class-based) returning an unsubscribe function from `subscribe`.
- Edge cases: multiple subscribers to the same topic, unsubscribing mid-publish, publishing to a topic with zero subscribers.

### C11. Simple analytics/event batching queue

A realistic "platform" style question: batch analytics events and flush them either when a batch size threshold is hit or a timer elapses (whichever first), and flush remaining events on page unload.

- Requirements: `track(eventName, payload)` queues an event; auto-flush when queue reaches `batchSize`; auto-flush every `flushIntervalMs` even if batch isn't full; expose `flush()` to send immediately (e.g. via `navigator.sendBeacon` conceptually).
- Edge cases: flush triggered by size vs by timer — timer should reset after any flush; flush with an empty queue should be a no-op; rapid `track()` calls right at the batch boundary.

### C12. Simple in-memory rate limiter (token bucket, conceptual)

- Requirements: `rateLimiter(maxCalls, windowMs)` returns a function that only allows `maxCalls` invocations within any rolling `windowMs` window, rejecting/queuing calls beyond that.
- Edge cases: burst of calls at the very start, calls spaced exactly at the window boundary, `maxCalls = 1`.

---

## How to use this list

1. Do **A4 (bind), A7 (Promise.all), C9 (EventEmitter), C1/C2 (debounce/throttle), and B5 (custom Promise)** first — these come up most often across Adobe-style rounds and cover the core mental models (closures, `this`, microtask/macrotask, async control flow).
2. For each: write it cold first (10-15 min), _then_ check your edge cases, _then_ say out loud how you'd explain your approach — that's the muscle they're actually testing in Round 1/2.
3. Once comfortable, time-box yourself to ~15 min per function to simulate interview pressure.

Want to work through any of these together right now — e.g. start with the EventEmitter (since it's already come up in a real Adobe round for you) or the custom Promise implementation?
