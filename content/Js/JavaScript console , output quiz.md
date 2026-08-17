---
{"publish":true,"created":"2026-08-12T10:44:02.200Z","modified":"2026-08-17T02:08:42.587Z"}
---

Format: read the code, predict the output (or spot the error), then check the answer. Answers are collapsed under each question — cover them with your hand/scroll slowly if you want the full interactive experience.

---

### Q1

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

**Answer: `3 3 3`** `var` is function-scoped, so all three callbacks close over the same `i`. By the time the callbacks run (after the loop finishes), `i` is `3`. Using `let` instead would print `0 1 2`, since `let` creates a new binding per iteration.

---

### Q2

```js
console.log(typeof null);
console.log(typeof undefined);
console.log(typeof NaN);
```

**Answer: `'object' 'undefined' 'number'`** `typeof null` is `'object'` (a long-standing JS bug kept for backward compatibility). `typeof undefined` is `'undefined'`. `NaN` is still a numeric value, so `typeof NaN` is `'number'`.

---

### Q3

```js
console.log([] == false);
console.log([] === false);
console.log('' == false);
```

**Answer: `true false true`** With `==`, `[]` is coerced: `[] -> '' -> 0`, and `false -> 0`, so `[] == false` is `true`. `===` skips coercion and compares types directly, so `[] === false` is `false` (array vs boolean). `'' == false`: `'' -> 0`, `false -> 0`, so `true`.

---

### Q4

```js
function outer() {
  let count = 0;
  return function inner() {
    count++;
    return count;
  };
}
const counter1 = outer();
const counter2 = outer();
console.log(counter1(), counter1(), counter2());
```

**Answer: `1 2 1`** Each call to `outer()` creates a fresh closure over its own `count` variable. `counter1` and `counter2` have independent counts, so `counter1()` twice gives `1, 2`, and `counter2()` starts fresh at `1`.

---

### Q5

```js
const obj = {
  name: 'Adobe',
  greet: function () {
    console.log(this.name);
  },
  greetArrow: () => {
    console.log(this.name);
  }
};
obj.greet();
obj.greetArrow();
```

**Answer: `'Adobe' undefined`** `greet` is a regular function, so `this` is bound to `obj` at call time → `'Adobe'`. `greetArrow` is an arrow function, so it captures `this` lexically from where `obj` was defined (module/global scope), not from `obj` → `this.name` is `undefined` there.

---

### Q6

```js
console.log('start');
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
console.log('end');
```

**Answer: `start end promise timeout`** Synchronous code runs first: `'start'`, then `'end'`. Then the microtask queue (Promise callbacks) drains before the macrotask queue (`setTimeout`), so `'promise'` logs before `'timeout'` even though both were scheduled with a 0ms-equivalent delay.

---

### Q7

```js
async function getValue() {
  return 42;
}
const result = getValue();
console.log(result);
```

**Answer: `Promise { 42 }`** An async function always returns a Promise, even if you `return` a plain value inside it. The value is wrapped: calling `getValue()` gives you `Promise { 42 }`, not `42` directly. You'd need `await getValue()` or `.then()` to unwrap it.

---

### Q8

```js
console.log(0.1 + 0.2 === 0.3);
console.log(0.1 + 0.2);
```

**Answer: `false, 0.30000000000000004`** Floating-point numbers are stored in binary, and `0.1` and `0.2` cannot be represented exactly, so their sum has a tiny rounding error.

---

### Q9

```js
const arr = [10, 2, 33, 4];
console.log(arr.sort());
```

**Answer: `[10, 2, 4, 33]`** `Array.prototype.sort()` with no comparator converts elements to strings and sorts lexicographically by default. `'10' < '2' < '33' < '4'` alphabetically. To sort numbers correctly you need `arr.sort((a, b) => a - b)`.

---

### Q10

```js
function Person(name) {
  this.name = name;
}
const p = Person('Anjul');
console.log(p);
```

**Answer: `undefined`** `Person` was called without `new`, so it's just a regular function call. `p` is `undefined` because `Person` returns nothing. In non-strict mode, `this.name = 'Anjul'` leaks onto the global object instead of creating a new instance.

---

### Q11

```js
const a = { x: 1 };
const b = a;
b.x = 2;
console.log(a.x);

const c = { x: 1 };
const d = { ...c };
d.x = 2;
console.log(c.x);
```

**Answer: `2, 1`** `b = a` copies the reference, not the object, so mutating `b.x` also changes `a.x` → `2`. Spreading with `{...c}` creates a shallow copy, a new object, so mutating `d.x` doesn't affect `c` → stays `1`.

---

### Q12

```js
function greet(name = 'stranger', greeting = `Hi ${name}`) {
  console.log(greeting);
}
greet();
greet(undefined, 'Hello');
```

**Answer: `'Hi stranger' 'Hello'`** Default parameters are evaluated left to right and can reference earlier parameters. `greet()` uses both defaults. `greet(undefined, 'Hello')` triggers the default for `name` too, but `greeting` is explicitly passed as `'Hello'`, overriding its default.

---

### Q13

```js
const user = null;
console.log(user?.profile?.email ?? 'no email');
```

**Answer: `'no email'`** Optional chaining (`?.`) short-circuits to `undefined` as soon as it hits a `null`/`undefined` value, instead of throwing. Then nullish coalescing (`??`) replaces that `undefined` with `'no email'`.

---

### Q14

```js
class Counter {
  count = 0;
  increment() {
    this.count++;
  }
}
const c = new Counter();
const fn = c.increment;
fn();
```

**Answer: `TypeError: Cannot read properties of undefined`** `increment` is a regular method, so its `this` is only bound when called as `c.increment()`. Once extracted into `fn` and called standalone, `this` is `undefined` (strict mode / ES modules), so `this.count++` throws. An arrow-function class field (`increment = () => {...}`) would fix this by binding `this` lexically.

---

### Q15

```js
function* generator() {
  yield 1;
  yield 2;
  return 3;
}
const gen = generator();
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
```

**Answer:**

```
{value: 1, done: false}
{value: 2, done: false}
{value: 3, done: true}
{value: undefined, done: true}
```

Each `next()` call resumes the generator until the next `yield`. The `return` statement produces the final `{value: 3, done: true}`. Any further `next()` calls after completion return `{value: undefined, done: true}`.

---

### Q16

```js
function test() {
  try {
    console.log('try');
    throw new Error('fail');
  } catch (e) {
    console.log('catch');
    return 'from catch';
  } finally {
    console.log('finally');
    return 'from finally';
  }
}
console.log(test());
```

**Answer: prints `try`, `catch`, `finally`, then `test()` returns `'from finally'`** The blocks execute in order, so all three logs print. But a `return` inside `finally` always overrides any earlier `return` (from `try` or `catch`) — the function ultimately returns `'from finally'`. Classic footgun; avoid `return` in `finally` in real code.

---

### Q17

```js
console.log(JSON.stringify({ a: undefined, b: function(){}, c: 1, d: Symbol('x') }));
```

**Answer: `'{"c":1}'`** `JSON.stringify` silently drops object properties whose value is `undefined`, a function, or a `Symbol` — they simply don't appear in the output. Only `c: 1` survives. (Note: inside an _array_, those same values become `null` instead of being dropped.)

---

### Q18

```js
const arr = [1, , 3];
console.log(arr.length);
console.log(arr.map(x => x * 2));
```

**Answer: `3, [2, empty, 6]`** `arr` has a length of `3` but index `1` is a real "hole" (sparse array), not a set `undefined` value. `Array.prototype.map` skips holes entirely rather than calling the callback on them, so the hole is preserved as-is in the output.

---

## Set 2 — new questions (medium-hard): closures, hoisting, promises, async/await, and beyond

### Q19 — hoisting

```js
console.log(a);
console.log(foo());
var a = 1;
function foo() {
  return 'hoisted';
}
console.log(b);
let b = 2;
```

<details><summary>Answer</summary>

`undefined`, `'hoisted'`, then a `ReferenceError: Cannot access 'b' before initialization`.

`var a` is hoisted and initialized to `undefined` before execution reaches it. Function declarations are fully hoisted (name _and_ body), so `foo()` works even before its definition line. `let b` is hoisted too, but stays in the "temporal dead zone" until its declaration line executes — accessing it before that throws, unlike `var`.

</details>

---

### Q20 — hoisting + function vs var

```js
var foo = 'outer';
function bar() {
  console.log(foo);
  var foo = 'inner';
  console.log(foo);
}
bar();
```

<details><summary>Answer</summary>

`undefined`, `'inner'`.

Inside `bar`, `var foo` is hoisted to the top of the function scope (shadowing the outer `foo`), but its _assignment_ stays in place. So the first `console.log(foo)` sees the hoisted-but-uninitialized local `foo` (`undefined`), not the outer `'outer'`. After the assignment, it becomes `'inner'`.

</details>

---

### Q21 — closures in a loop, IIFE fix

```js
const funcs = [];
for (var i = 0; i < 3; i++) {
  funcs.push(function () {
    return i;
  });
}
console.log(funcs.map(f => f()));

const funcs2 = [];
for (var j = 0; j < 3; j++) {
  (function (captured) {
    funcs2.push(function () {
      return captured;
    });
  })(j);
}
console.log(funcs2.map(f => f()));
```

<details><summary>Answer</summary>

`[3, 3, 3]` then `[0, 1, 2]`.

First loop: all closures share the same `var i`, which is `3` by the time any function runs. Second loop: the IIFE captures the _current value_ of `j` into a new parameter `captured` on each iteration, giving each pushed function its own snapshot — the classic pre-ES6 fix for the `let` problem.

</details>

---

### Q22 — closures and private state

```js
function makeBankAccount(balance) {
  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) throw new Error('insufficient funds');
      balance -= amount;
      return balance;
    },
    getBalance: () => balance
  };
}
const acc1 = makeBankAccount(100);
const acc2 = makeBankAccount(50);
acc1.deposit(20);
acc2.withdraw(10);
console.log(acc1.getBalance(), acc2.getBalance());
console.log(acc1.balance);
```

<details><summary>Answer</summary>

`120 40`, then `undefined`.

Each call to `makeBankAccount` creates a new closure over its own `balance` variable — this is the module pattern for private state. `acc1.balance` is `undefined` because `balance` was never exposed as a property; it only exists in the closure, accessible solely through the returned methods.

</details>

---

### Q23 — promise chaining and return values

```js
Promise.resolve(1)
  .then(val => {
    console.log(val);
    return val + 1;
  })
  .then(val => {
    console.log(val);
    throw new Error('fail at 2');
  })
  .catch(err => {
    console.log('caught:', err.message);
    return 100;
  })
  .then(val => {
    console.log('final:', val);
  });
```

<details><summary>Answer</summary>

```
1
2
caught: fail at 2
final: 100
```

Each `.then()` passes its return value to the next `.then()` in the chain. A thrown error skips all subsequent `.then()` handlers until it hits a `.catch()`. Once `.catch()` handles it and returns a value (`100`), the chain resumes normally as fulfilled from that point forward.

</details>

---

### Q24 — microtask vs macrotask ordering (harder)

```js
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve()
  .then(() => console.log('3'))
  .then(() => console.log('4'));

queueMicrotask(() => console.log('5'));

console.log('6');
```

<details><summary>Answer</summary>

`1 6 3 5 4 2`

Synchronous code runs first: `1`, `6`. Then the microtask queue drains fully before the next macrotask: the first `.then` (`3`) was queued before `queueMicrotask`'s callback (`5`), so `3` runs before `5` — but the _second_ `.then` (`4`) is only queued once the first one resolves, which happens after `5` was already queued, so `4` runs after `5`. `setTimeout` (`2`) is a macrotask and always runs last, after all microtasks are exhausted.

</details>

---

### Q25 — async/await and the event loop

```js
async function foo() {
  console.log('foo start');
  await null;
  console.log('foo end');
}

console.log('script start');
foo();
console.log('script end');
```

<details><summary>Answer</summary>

`script start`, `foo start`, `script end`, `foo end`.

Calling `foo()` runs synchronously until the first `await`. `console.log('foo start')` fires immediately. `await null` pauses `foo`, scheduling the rest of the function as a microtask (even though `null` isn't a real promise, `await` always yields at least one microtask tick). Control returns to the caller, so `'script end'` logs before the resumed `'foo end'`.

</details>

---

### Q26 — async/await inside loops: sequential vs parallel

```js
async function delay(val, ms) {
  return new Promise(resolve => setTimeout(() => resolve(val), ms));
}

async function sequential() {
  const results = [];
  for (const ms of [300, 100, 200]) {
    results.push(await delay(ms, ms));
  }
  return results;
}

async function parallel() {
  const promises = [300, 100, 200].map(ms => delay(ms, ms));
  return Promise.all(promises);
}
```

<details><summary>Answer</summary>

`sequential()` takes roughly `300 + 100 + 200 = 600ms` total, resolving with `[300, 100, 200]` (order preserved because it's pushed in loop order, and each `await` blocks until that iteration's promise resolves before starting the next).

`parallel()` takes roughly `300ms` total (the longest single delay), because all three `delay()` calls start immediately, running concurrently, and `Promise.all` waits for the slowest one. It still resolves with `[300, 100, 200]` — `Promise.all` preserves input order regardless of completion order.

This is one of the most common real-world async/await mistakes: using `await` inside a `for` loop when the operations don't depend on each other, needlessly serializing what could run in parallel.

</details>

---

### Q27 — unhandled rejection timing

```js
async function risky() {
  throw new Error('boom');
}

risky();
console.log('after call');
```

<details><summary>Answer</summary>

Prints `'after call'`, and separately (asynchronously) an unhandled promise rejection warning/error for `'boom'`.

`risky()` returns a rejected promise, but since nothing calls `.catch()` or `await`s it, the rejection isn't handled synchronously. The `console.log` runs immediately after, and the unhandled rejection surfaces afterward (in Node, as an `unhandledRejection` event; in browsers, as a console warning) — it does not stop or throw synchronously in the calling code.

</details>

---

### Q28 — closures + setTimeout + block scope combined

```js
const arr = [];
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    arr.push(i);
    if (arr.length === 3) console.log(arr);
  }, i * 10);
}
```

<details><summary>Answer</summary>

`[0, 1, 2]`

With `let`, each iteration of the loop creates a fresh binding of `i` scoped to that iteration, so each `setTimeout` callback closes over its own `i`. Combined with the increasing delays (`0, 10, 20`ms), they also happen to fire in order, producing `[0, 1, 2]` — though the _order_ of pushes is guaranteed by the delays here, not just by `let` alone (all three would still each have their correct own `i` regardless of timing).

</details>

---

### Q29 — `this` inside a `setTimeout` inside a method (mixing concepts)

```js
const obj = {
  value: 42,
  delayedLog: function () {
    setTimeout(function () {
      console.log(this.value);
    }, 0);
    setTimeout(() => {
      console.log(this.value);
    }, 0);
  }
};
obj.delayedLog();
```

<details><summary>Answer</summary>

`undefined`, then `42`.

The first `setTimeout` uses a regular function, whose `this` is determined by how _it's_ called — `setTimeout` invokes it as a plain function, so `this` is the global object (or `undefined` in strict mode) → `this.value` is `undefined`. The second uses an arrow function, which lexically inherits `this` from `delayedLog`'s scope — where `this` is `obj` → `42`.

</details>

---

### Q30 — Promise executor runs synchronously

```js
console.log('1');
new Promise((resolve) => {
  console.log('2');
  resolve();
  console.log('3');
}).then(() => console.log('4'));
console.log('5');
```

<details><summary>Answer</summary>

`1 2 3 5 4`

The executor function passed to `new Promise(...)` runs **synchronously**, immediately, at the point of construction — it's not deferred. So `'2'` and `'3'` log before `'5'`. Only the `.then()` callback is deferred to the microtask queue, running after all synchronous code (`'5'`) finishes.

</details>

---

### Q31 — closures capturing by reference vs primitive copy (object mutation trap)

```js
function createLoggers(users) {
  return users.map(user => () => console.log(user.name));
}

const users = [{ name: 'A' }, { name: 'B' }];
const loggers = createLoggers(users);
users[0].name = 'Changed';
loggers[0]();
loggers[1]();
```

<details><summary>Answer</summary>

`'Changed'`, `'B'`

Each closure captures a reference to the _object_ `user`, not a snapshot of `user.name`. Since objects are captured by reference, mutating `users[0].name` after the loggers were created still affects what `loggers[0]` sees when it later reads `user.name`. This differs from primitive closures (like the earlier `var i` examples), where the _value_ itself, not a mutable object, is what's shared or copied.

</details>

---

### Q32 — `Promise.race` and `Promise.any` distinction

```js
const p1 = new Promise((_, reject) => setTimeout(() => reject('err-fast'), 50));
const p2 = new Promise((resolve) => setTimeout(() => resolve('ok-slow'), 150));

Promise.race([p1, p2]).then(console.log).catch(e => console.log('race caught:', e));
Promise.any([p1, p2]).then(console.log).catch(e => console.log('any caught:', e));
```

<details><summary>Answer</summary>

`race caught: err-fast` (around 50ms), then `ok-slow` (around 150ms).

`Promise.race` settles as soon as _any_ promise settles — even if it's a rejection — so it rejects fast with `'err-fast'`. `Promise.any` ignores rejections and waits for the first _fulfillment_; it only rejects if _all_ promises reject (with an `AggregateError`). Here it waits past the rejection and resolves once `p2` fulfills at ~150ms with `'ok-slow'`.

</details>

---

### Q33 — async function returning a promise vs returning a value nested

```js
async function inner() {
  return Promise.resolve('inner value');
}
async function outer() {
  const result = await inner();
  console.log(result);
  return inner();
}
outer().then(val => console.log('outer resolved with:', val));
```

<details><summary>Answer</summary>

`'inner value'`, then `'outer resolved with: inner value'`.

`await inner()` unwraps the promise-returning-a-promise down to the actual value `'inner value'`, since `await` always fully unwraps, even nested thenables. `outer()` then `return inner()` — returning a promise from an async function doesn't double-wrap it; the outer promise adopts the state of the returned inner promise, so `.then()` on `outer()` still receives the plain unwrapped value.

</details>

---

## Suggested drill order

1. Re-do Set 1 cold (no answers visible) — target: explain _why_, not just the output, in under 20 seconds per question.
2. Move to Set 2, focusing especially on **Q24, Q26, Q30, Q32, Q33** — these are the ones most likely to come up as follow-up "why" questions after a live coding exercise (interviewers love probing async ordering once they see you write a `Promise.all` or `async/await` snippet).
3. For any question you get wrong, write a 2-3 line explanation in your own words — that's usually a stronger signal of readiness than just re-reading the given explanation.
