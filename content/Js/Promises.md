---
{"publish":true,"created":"2026-04-22T12:46:53.466Z","modified":"2026-08-17T02:08:55.949Z"}
---

# JavaScript Promises — Complete Reference

---

**let data = Await somePromise() if promise rejects then awaits throw wihout assiging any value to data**

## What Is a Promise?

A Promise represents the **eventual completion or failure** of an async operation. It's JS's clean solution to callback hell — instead of deeply nested callbacks, you chain `.then()` calls flatly.

---

## Promise States

A promise moves in one direction only — never back:

text

`pending → fulfilled (has a value)         ↘ rejected  (has a reason/error)`

Once settled, a promise is **immutable** — calling `resolve` or `reject` again is silently ignored:

js

```
const p = new Promise((resolve, reject) => {   
resolve("first");  // ✅ settles the promise  
resolve("second"); // ❌ silently ignored  
reject("error");   // ❌ silently ignored 
}); 

p.then(val => console.log(val)); // always "first"
```

---

## Creating a Promise

js

```
const p = new Promise((resolve, reject) => {   // executor — runs SYNCHRONOUSLY and immediately  if (success) resolve("data");  else reject(new Error("failed")); });
```

---

## Consuming — `.then()`, `.catch()`, `.finally()`

js

`p   .then(val => console.log(val))    // runs if fulfilled  .catch(err => console.error(err)) // runs if rejected  .finally(() => console.log("always runs"));`

- `.catch()` is shorthand for `.then(null, onRejected)`

- `.finally()` runs regardless of outcome, receives no value

---

## Promise Chaining

Each `.then()` returns a **new Promise**, enabling flat chaining:

js

``fetch("/api/user")   .then(res => res.json())          // returns a new promise — chain waits  .then(user => user.id)            // returns primitive — chain continues immediately  .then(id => fetch(`/api/posts/${id}`)) // returns promise — chain waits again  .then(res => res.json())  .catch(err => console.error(err)); // catches ANY error anywhere in chain``

## `.then()` Return Value Rules

|`.then()` returns|What happens next|
|---|---|
|**Primitive** (`10`, `"hi"`)|Wrapped in `Promise.resolve()` — next `.then()` runs immediately|
|**A Promise**|Chain **waits** for it to settle|
|**Nothing**|Treated as `Promise.resolve(undefined)` — next gets `undefined`|
|**Throws**|Chain skips to nearest `.catch()`|

## When a `.then()` Rejects

All subsequent `.then()` blocks are **skipped** — chain jumps to nearest `.catch()`. After `.catch()` handles the error, the chain **resumes**:

js

`Promise.resolve("start")   .then(() => { throw new Error("oops"); }) // ❌ rejects here  .then(() => console.log("skipped"))        // ❌ skipped  .catch(err => { console.log("caught"); return "recovered"; }) // ✅ runs  .then(val => console.log(val));            // ✅ "recovered" — chain resumed`

---

## Chaining — Primitive vs Promise Return

js

`function delay(val) {   return new Promise(res => setTimeout(() => res(val), 1000)); } Promise.resolve(5)   .then(val => val * 2)           // returns 10 (primitive) — immediate  .then(val => delay(val + 5))    // returns Promise — chain WAITS 1s  .then(val => val * 3)           // gets 45 — immediate  .then(val => console.log(val)); // 45`

---

## Sequential & Non-Blocking

- Each `.then()` waits for the previous to resolve — **sequential within the chain**

- But the **main thread is never blocked** — code outside the chain continues normally

js

`fetchSomething()   .then(step1)  // waits for fetchSomething  .then(step2); // waits for step1 console.log("runs immediately, doesn't wait"); // ✅ non-blocking`

---

## Static Methods

|Method|Behaviour|
|---|---|
|`Promise.all([p1,p2])`|Waits for ALL to fulfill; **fail-fast** — rejects on first rejection|
|`Promise.allSettled([p1,p2])`|Always returns N results for N promises — `{ status, value/reason }`|
|`Promise.race([p1,p2])`|Returns the **first** result — fulfilled OR rejected|
|`Promise.any([p1,p2])`|Returns the **first fulfilled** result; rejects only if ALL reject (`AggregateError`)|

---

## `Promise.resolve()` — Syntax Deep Dive

js

`Promise.resolve(value)`

|Value passed|Result|
|---|---|
|Primitive / object|New fulfilled Promise wrapping it|
|Existing native Promise|**Same promise returned** — no double wrapping|
|Thenable (has `.then()`)|New Promise that follows thenable's state|
|Rejected Promise|Still rejected — `resolve` ≠ "make it succeed"|
|Nested Promises|**Flattened** to single layer|

**Real-world uses** (not just for learning!):

- Normalise values that may or may not be a Promise

- Return early from functions that must always return a Promise (e.g. cache hits)

- Mix plain values with promises in `Promise.all([])`

---

## `async`/`await` — Syntactic Sugar Over Promises

`async`/`await` doesn't replace promises — it's just cleaner syntax. Under the hood, still promises.

js

`async function loadData() {   try {    const res = await fetch("/api/user"); // pauses HERE inside this function only    const user = await res.json();    console.log(user);  } catch (err) {    console.error(err);  } } loadData(); console.log("runs immediately"); // ✅ not blocked`

`await` pauses **only the current async function** — the rest of the program keeps running.

---

## Microtask Queue — Never "Inline"

`.then()`/`.catch()` callbacks are **always deferred** — even if the promise is already resolved, they never run synchronously in the current execution flow:

js

`const p = Promise.resolve("hello"); // already resolved p.then(val => console.log(val)); // queued in microtask queue console.log("runs first"); // sync, runs inline // Output: // "runs first" // "hello"`

**Microtasks (Promises) always run before Macrotasks (setTimeout):**

js

`setTimeout(() => console.log("macrotask"), 0); Promise.resolve().then(() => console.log("microtask")); console.log("sync"); // Output: // "sync" // "microtask"  ← promise runs before setTimeout // "macrotask"`

---

## Grand Mental Model

text

`new Promise(executor)     ← runs synchronously        │       ├── resolve(val)   → fulfilled → .then(cb) → microtask queue       └── reject(err)    → rejected  → .catch(cb) → microtask queue                                               │                              call stack empties → microtasks flush                                               │                                        .finally() always last`
