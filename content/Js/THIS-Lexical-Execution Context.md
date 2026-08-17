---
{"publish":true,"created":"2025-12-19T17:38:01.000Z","modified":"2026-08-17T02:08:59.870Z"}
---

# Complete Summary: Execution Context, Scope, and `this`

## The Big Picture: Two Separate Timelines

1. **Parsing (compilation)**: happens once, before any code executes. Static analysis only, no memory allocation. The engine reads the whole script, checks syntax, and locates where all declarations (`var`, `let`, `const`, functions) are. This pass is what makes hoisting possible later, but parsing itself is not hoisting.

2. **Execution Context (EC) creation and run**: happens every time a piece of code is about to execute, once for global code, and fresh again for every single function call. Creating an EC has two internal phases:

   - **Creation phase**: this is where hoisting actually happens
   - **Execution phase**: code runs top to bottom

So the correct order is: parse everything once, then for each EC, creation phase runs first, then execution phase.

## What an EC Actually Contains

An EC has **three separate components**, siblings, not nested inside each other:

```
Execution Context
├── Variable Environment   (var, function declarations)
├── Lexical Environment    (let, const, + outer reference)
└── this binding           (separate slot, set at call-time)
```

### Variable Environment (VE)

- Holds `var` declarations and function declarations
- Function-scoped: ignores `{}` blocks (`if`, `for`) entirely, always lands in the nearest enclosing function's VE
- Set up during creation phase: `var`s initialized to `undefined`, functions fully hoisted and immediately callable

### Lexical Environment (LE)

- Holds `let`/`const` declarations
- Block-scoped: every `{}` block gets its own fresh LE
- Each LE carries an `outer` reference to its parent LE, fixed at write-time (where the code is written), never changes based on where it's called from
- `let`/`const` are hoisted too, but left uninitialized in the TDZ until their line actually executes

### `this` binding

- A separate slot, not part of the Lexical Environment
- Set during creation phase, but its value depends on call-time behavior for regular functions
- Arrow functions do not get this slot created at all

| How called                                   | `this` value                    |
| -------------------------------------------- | ------------------------------- |
| `new foo()`                                  | Newly created object            |
| `foo.call(obj)` / `apply(obj)` / `bind(obj)` | Explicitly passed `obj`         |
| `obj.method()`                               | `obj` (object before the dot)   |
| Plain `foo()` in strict mode                 | `undefined`                     |
| Plain `foo()` non-strict                     | `globalThis` / `window`         |
| Arrow function                               | Lexically inherited (see below) |

## Clarifications Confirmed Along the Way

**Can one EC have multiple Variable Environments?** No. One EC, one VE, since VE is function-scoped by definition. There is nothing that would create a second one within the same EC.

**Does the "child scope invisible to parent" rule apply to `var` the same way as `let`/`const`?** Two different boundaries need to stay separate:

- Within the same function, `var` ignores block boundaries (`if`, `for`, `{}`) and leaks into the function's VE. `let`/`const` stay trapped in the block's LE.
- Across function boundaries, `var` behaves exactly like `let`/`const`: a parent function can never access a child function's variables, regardless of whether they were declared with `var`, `let`, or `const`. `var`'s block-escaping trick only works inside a single function, it has no power to escape the function itself.

**`this` in regular functions** Confirmed correct: each call to a function creates a new EC, and each EC gets its own independently resolved `this` binding, based on how that specific call was made (`new`, `.call/.apply/.bind`, `obj.method()`, bare call). Calling the same function multiple different ways produces different `this` values across separate ECs.

**`this` in arrow functions** Confirmed correct, with the precise mechanism nailed down: an arrow function's EC is created like any other (it has its own VE and LE for local variables), but it has no `this` slot at all. When `this` is referenced inside an arrow, the engine cannot find it locally, so it falls back to walking the **scope chain**, the same chain of `outer` LE references used to resolve any ordinary unresolved variable, until it reaches an EC that does have a populated `this` slot. That EC necessarily belongs to the nearest enclosing regular function (or global). This is not a separate special `this`-only path, it is literally the same lookup mechanism used for any variable that isn't found locally. There is no alternate "somewhere else" it searches, it is the scope chain, full stop.

**Why `bind`/`call`/`apply` don't work on arrow functions** Since there's no `this` slot to override in an arrow's EC, `.call(obj)`, `.apply(obj)`, `.bind(obj)` silently ignore the `this` argument, though positional arguments still pass through normally. The only way to influence what `this` an arrow resolves to is to control the `this` of whichever enclosing regular function the arrow is nested inside.

Here's a set of snippets moving from basic to genuinely tricky, each isolating one specific behavior.

## 1. Same function, different call styles, different `this`

```js
function show() {
  console.log(this);
}

const obj = { show };

show();          // globalThis (non-strict) or undefined (strict)
obj.show();      // obj, because it's called as obj.show()

const detached = obj.show;
detached();      // globalThis / undefined, this is LOST, obj is no longer before the dot
```

**Lesson**: `this` is not baked into the function. It's decided fresh at each call-site, based purely on syntax to the left of the `()`, not on where the function was defined or previously attached.

## 2. Losing `this` in a callback, the classic bug

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(function () {
      this.seconds++;               // this is NOT timer here
      console.log(this.seconds);    // NaN, this is globalThis/undefined
    }, 1000);
  }
};
timer.start();
```

The function passed to `setInterval` is invoked bare by the timer internals, `setInterval(fn)`, not `setInterval(obj.fn)`. So the call-site rule kicks in and `this` is not `timer`.

## 3. Fixing it with an arrow function

```js
const timer2 = {
  seconds: 0,
  start() {
    setInterval(() => {
      this.seconds++;                // this is inherited from start()'s this
      console.log(this.seconds);     // works correctly
    }, 1000);
  }
};
timer2.start();
```

The arrow has no `this` of its own, so it walks up the scope chain to `start()`'s EC, where `this` was set to `timer2` because `start()` was called as `timer2.start()`.

## 4. Arrow function as an object method, a trap

```js
const obj2 = {
  name: "Anjul",
  greet: () => {
    console.log(this.name);   // NOT "Anjul"
  }
};
obj2.greet();  // undefined, object literals don't create a this binding
```

Even though this arrow looks like it's "inside" `obj2`, object literal `{}` syntax does not create an execution context or a `this` binding. The arrow's outer scope is wherever `obj2` itself was written, usually the module or global scope. It never sees `obj2` as `this`.

## 5. `bind` on a regular function vs an arrow function

```js
function regularFn() {
  console.log(this.name);
}
const arrowFn = () => {
  console.log(this.name);
}

const person = { name: "Anjul" };

const boundRegular = regularFn.bind(person);
boundRegular();   // "Anjul", bind works on regular functions

const boundArrow = arrowFn.bind(person);
boundArrow();     // ignores the bind, this comes from wherever arrowFn was defined
```

## 6. Constructor calls (`new`) vs arrow functions used as constructors

```js
function Person(name) {
  this.name = name;
}
const p = new Person("Anjul");
console.log(p.name);  // "Anjul", this is the newly created object

const ArrowPerson = (name) => {
  this.name = name;
};
// new ArrowPerson("Anjul");  // TypeError: ArrowPerson is not a constructor
```

Arrow functions have no `this` binding to hijack for a new object, so JS disallows using them with `new` entirely, it throws immediately rather than silently doing the wrong thing.

## 7. `this` inside nested regular functions, does not inherit like arrows do

```js
const obj3 = {
  name: "Anjul",
  outer() {
    console.log(this.name);   // "Anjul", called as obj3.outer()

    function inner() {
      console.log(this.name); // undefined, inner() called bare inside outer()
    }
    inner();
  }
};
obj3.outer();
```

Unlike arrows, a nested **regular** function gets its own fresh `this` binding, determined by how `inner()` itself is called, which here is a bare call. It does not inherit `outer`'s `this` just because it's lexically nested inside it. This is precisely the contrast that makes arrow functions useful, regular functions reset `this` at every call boundary, arrows never do.

## 8. Mixing both in the same object, side by side

```js
const obj4 = {
  name: "Anjul",
  regularMethod() {
    console.log("regular:", this.name);           // "Anjul"

    function innerRegular() {
      console.log("inner regular:", this?.name);  // undefined, this resets
    }
    innerRegular();

    const innerArrow = () => {
      console.log("inner arrow:", this.name);      // "Anjul", inherits from regularMethod
    };
    innerArrow();
  }
};
obj4.regularMethod();

// Output:
// regular: Anjul
// inner regular: undefined
// inner arrow: Anjul
```

This single snippet is a good one to keep as a reference. Same object, same outer method, but the nested regular function resets `this` on its own call, while the nested arrow function reaches back through the scope chain to reuse `regularMethod`'s `this`.

## 9. Class methods, a real-world variant of the same bug

```js
class Counter {
  count = 0;

  incrementRegular() {
    this.count++;
  }

  incrementArrow = () => {
    this.count++;
  }
}

const c = new Counter();

const detachedRegular = c.incrementRegular;
const detachedArrow = c.incrementArrow;

// detachedRegular();  // TypeError: Cannot read properties of undefined, this is lost
detachedArrow();       // works fine, arrow captured this from the constructor's context
```

## 10. Bare Assignment vs `var` — The Mutation Gotcha

```
// Snippet 1: var creates a LOCAL copy
var x = 1;

function a() {
  var x = 2; // new local x, global x untouched
  b();
}

function b() {
  console.log(x);
}

a(); // logs 1


// Snippet 2: bare assignment mutates global
x = 1;

function a() {
  x = 2; // walks scope chain, mutates global x
  b();
}

function b() {
  console.log(x);
}

a(); // logs 2

```

---

This is why arrow-function class fields (`incrementArrow = () => {}`) became a common pattern for event handlers and callbacks, in React especially, they survive being detached and passed around, since the arrow's `this` was permanently fixed to the instance when the field was created, not re-derived at each call.

**The single unifying rule across all nine examples**: for regular functions, ask "what's immediately to the left of the dot at the call-site." For arrow functions, ignore the call-site entirely and instead ask "what `this` was active in the nearest enclosing regular function, at the point this arrow was written."
