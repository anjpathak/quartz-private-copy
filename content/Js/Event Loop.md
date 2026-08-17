---
{"publish":true,"created":"2026-04-22T11:38:00.236Z","modified":"2026-08-17T02:08:25.728Z"}
---

Here's a crisp summary of everything we covered:

---

## JavaScript Event Loop — Discussion Summary

## Core Architecture

JavaScript is single-threaded. The event loop coordinates between:

- **Call stack** — runs all synchronous code (LIFO)

- **Web APIs** — handle async work outside the JS engine (`setTimeout`, `fetch`, etc.)

- **Microtask queue** — `Promise.then`, `queueMicrotask`, `MutationObserver`

- **Macrotask queue** — `setTimeout`, `setInterval`, I/O, UI events

---

## The Loop Algorithm

text

`1. Run current sync code on call stack to completion 2. Drain entire microtask queue (including newly added microtasks) 3. Maybe render (browser) 4. Pick ONE macrotask → run it 5. Repeat from step 2`

Key asymmetry: **microtask queue is fully drained each tick; macrotask queue gives up only one task per tick.**

---

## Both Queues Only Hold Ready Callbacks

This was a key nuance we discussed:

- **A Promise object never sits in the microtask queue** — only its `.then` callback does, and only _after_ the promise has already settled

- Calling `resolve()` is what _pushes_ the callback into the microtask queue — it's **push-based, not pull-based**

- Same rule for macrotasks — a `setTimeout` callback only enters the macrotask queue _after_ the timer has expired

So **nothing pending ever waits in either queue** — presence in a queue = ready to execute.

js

`// p is pending → .then callback is NOT yet in microtask queue const p = new Promise(resolve => setTimeout(resolve, 2000)); p.then(() => console.log('runs after 2s')); // Event loop doesn't wait — it moves on immediately // Only after 2s does resolve() fire → callback enters microtask queue`

---

## Microtask Spawned Inside a Macrotask Runs Before the Next Macrotask

js

`setTimeout(() => {   console.log('macrotask 1');  Promise.resolve().then(() => console.log('microtask inside macrotask 1')); }, 0); setTimeout(() => console.log('macrotask 2'), 0); // Output: macrotask 1 → microtask inside macrotask 1 → macrotask 2`

Even though `macrotask 2` was already sitting in the queue ready to go, the microtask gets priority.

---

## The Event Loop Never Interrupts a Running Call Stack

"Run to completion" is the guarantee:

- The event loop **only pushes to the call stack when it's completely empty**

- Sync code called inside a running task (nested function calls) is purely the call stack's internal business — normal LIFO frames

- Even if a timer expires or a Promise resolves mid-execution, their callbacks just queue up and wait

- **Nothing external can inject into a running call stack** — no preemption, ever

---

## Notable Caveats

- `setTimeout(fn, 0)` still fires _after_ all microtasks, and has a ~4ms minimum in browsers after 5 levels of nesting

- An infinite microtask loop (`Promise.resolve().then(self)`) starves macrotasks and blocks rendering forever

- Each `await` in `async/await` is a microtask suspension — code after `await` is a microtask continuation

- In Node.js, `process.nextTick` fires before even Promise microtasks — it has the highest priority
