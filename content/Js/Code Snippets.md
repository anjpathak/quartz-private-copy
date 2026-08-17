---
{"publish":true,"created":"2026-08-12T09:55:03.182Z","modified":"2026-08-16T11:34:45.005Z"}
---

#### Async / Await

```js
async function getValue() {
  return 42;
}
const result = getValue();
console.log(result);
```

An async function always returns a Promise, even if you 'return' a plain value inside it. The value is wrapped. Calling getValue() gives you Promise { 42 }, not 42 directly. You'd need await getValue() or .then() to unwrap it.
