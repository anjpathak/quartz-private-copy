---
{"publish":true,"created":"2026-01-17T17:49:54.000Z","modified":"2026-08-17T03:03:47.499Z"}
---

# React Hooks — Explained Through Production Problems

For each hook: the real problem → what people naively try first → why that fails → why the hook is the right tool.

---

## 1. useRef — "Focus this input when the page loads"

**Scenario:** Login page. On mount, the email input should already be focused so the user can start typing immediately.

**Option A — grab the DOM node manually**

```javascript
document.getElementById('email').focus();
```

Problems:

- Runs during React's render phase, but the actual `<input>` DOM node isn't guaranteed to exist in the browser yet — React hasn't committed it. You'd get `null` or focus a stale node.
- IDs aren't safe in React. If this component renders twice on the same page (e.g. login form in a modal AND a page), you get duplicate IDs — classic bug.
- It's an imperative call sitting inside a declarative render function. It fights React's model instead of working with it.

**Option B — `useRef` + `useEffect`**

```javascript
function LoginForm() {
  const emailRef = useRef(null);

  useEffect(() => {
    emailRef.current.focus(); // guaranteed to exist here — runs after commit
  }, []);

  return <input ref={emailRef} type="email" />;
}
```

**Why this wins:** `useRef` gives you a mutable box that (a) persists across renders without causing new ones, and (b) React itself populates with the _actual DOM node_ once it's committed to the page. `useEffect` guarantees your code runs _after_ that commit. Together they solve: _"I need to imperatively touch the real DOM without breaking React's re-render cycle or relying on fragile selectors."_

**The core problem `useRef` solves:** React re-renders wipe out normal variables and don't give you direct DOM access. `useRef` is the sanctioned escape hatch for state that needs to survive renders _but shouldn't trigger a re-render when it changes_ (DOM nodes, timers, previous values, mutable flags).

---

## 2. useState — "Let the user change the quantity in a cart item"

**Scenario:** A product card has a quantity stepper. Clicking `+` should visually update the number.

**Option A — a plain variable**

```javascript
function ProductCard() {
  let quantity = 1;
  return <button onClick={() => quantity++}>{quantity}</button>;
}
```

Problem: `quantity++` changes the variable, but React has no idea it changed — nothing tells it to re-render. The UI is frozen even though the data changed underneath.

**Option B — a variable outside the component (module scope)** Now every `<ProductCard>` on the page (e.g. in a product grid) shares the _same_ quantity — clicking `+` on one card bumps them all. State leaks across instances.

**Option C — `useState`**

```javascript
const [quantity, setQuantity] = useState(1);
```

**Why this wins:** `useState` gives each component _instance_ its own isolated slot of memory, and — critically — calling `setQuantity` explicitly tells React "re-render this component with the new value." That's the missing link in option A.

**The core problem `useState` solves:** JS variables don't survive re-renders (they reset) and changing them doesn't trigger new renders. `useState` does both — persists per-instance, and re-renders on change.

---

## 3. useEffect — "Load this user's profile when the page opens"

**Scenario:** `/users/42` should fetch and display user #42's data on load, and refetch if the URL param changes.

**Option A — fetch directly in the component body**

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  fetch(`/api/users/${userId}`).then(r => r.json()).then(setUser); // ❌
  return <div>{user?.name}</div>;
}
```

Problem: This runs on _every single render_, including the one caused by `setUser` itself → infinite fetch loop. It also runs during the render phase, which React expects to be pure (no side effects, no network calls) — this can cause tearing and duplicate requests, especially in Strict Mode / concurrent rendering.

**Option B — fetch only inside a button's `onClick`** Works for "load on demand," but not for "load automatically when the page opens" — there's no click to hook into.

**Option C — `useEffect`**

```javascript
useEffect(() => {
  let cancelled = false;
  fetch(`/api/users/${userId}`)
    .then(r => r.json())
    .then(data => { if (!cancelled) setUser(data); });
  return () => { cancelled = true; }; // cleanup: avoids setting state on a stale/unmounted request
}, [userId]);
```

**Why this wins:** `useEffect` runs _after_ the render is committed, so it can't cause the render-loop from Option A. The dependency array (`[userId]`) tells React exactly when to re-run it — on mount, and again only when `userId` changes. The cleanup function solves a subtler bug: if `userId` changes quickly (user navigates 42 → 43), the response for 42 might arrive _after_ 43's request — without cleanup, you'd briefly show the wrong user's data.

**The core problem `useEffect` solves:** synchronizing a component with something _outside_ React (API, subscription, timer, DOM) without violating React's rule that rendering must be a pure calculation.

---

## 4. useContext — "The logged-in user's info is needed in the header, sidebar, and a deeply nested settings button"

**Scenario:** Auth state (`user`, `logout()`) is needed in components 4-5 levels deep in the tree.

**Option A — prop drilling**

```javascript
<App user={user}>
  <Layout user={user}>
    <Sidebar user={user}>
      <SettingsPanel user={user}>
        <LogoutButton user={user} /> {/* finally used here */}
```

Problem: `Layout`, `Sidebar`, `SettingsPanel` don't care about `user` at all — they're just relaying it. Every time you add a field to `user`, you touch every file in the chain. Refactoring becomes painful and error-prone.

**Option B — a global variable outside React (e.g. `window.currentUser`)** Not reactive — if the user logs out, nothing tells React to re-render components reading that variable. UI goes stale.

**Option C — `useContext`**

```javascript
const AuthContext = createContext();
// wrap app: <AuthContext.Provider value={{ user, logout }}>

function LogoutButton() {
  const { logout } = useContext(AuthContext); // pulled directly, no drilling
}
```

**Why this wins:** Any component, at any depth, can subscribe directly to the context value — and React _does_ re-render subscribers when the Provider's value changes, unlike Option B.

**The core problem `useContext` solves:** passing data through many layers of components that don't actually need it themselves, purely to relay it downward.

---

## 5. useReducer — "A multi-field checkout form where fields and errors update together"

**Scenario:** Checkout form: email, address, card number, plus validation errors and a submitting flag — several pieces of state that change _together_ as a unit (e.g. "start submit" should set `isSubmitting: true` AND clear old errors in one atomic step).

**Option A — separate `useState` calls per field**

```javascript
const [email, setEmail] = useState('');
const [errors, setErrors] = useState({});
const [isSubmitting, setIsSubmitting] = useState(false);
```

Problem: Works fine for simple, independent fields — but once an action needs to update _several of these together based on the current state of others_, the update logic gets scattered across many handlers, and you risk one update firing without the other (e.g. setting `isSubmitting: true` but forgetting to clear a stale error), leaving state briefly inconsistent.

**Option B — `useReducer`**

```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true, errors: {} };
    case 'SUBMIT_ERROR':
      return { ...state, isSubmitting: false, errors: action.errors };
  }
}
const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: 'SUBMIT_START' });
```

**Why this wins:** All the "what changes together" logic lives in one function, as explicit, testable transitions. Components just describe _what happened_ (`dispatch({type: 'SUBMIT_START'})`) rather than manually orchestrating five `setX` calls in the right order.

**The core problem `useReducer` solves:** coordinating multiple pieces of related state that must update consistently together, especially when the next state depends on the current state in a non-trivial way.

---

## 6. useCallback — "Typing in a search box shouldn't re-render a 10,000-row list"

**Scenario:** A search input sits above a heavy, memoized `<ResultsList />` (wrapped in `React.memo`). Typing should only re-run the list's render when results actually change — not on every keystroke.

**Option A — pass an inline function as a prop**

```javascript
<ResultsList onSelect={(id) => setSelected(id)} />
```

Problem: Every render of the parent creates a _brand-new function_. Even though `React.memo` on `ResultsList` checks "did props change?", a new function reference counts as a change every time — so `React.memo` never gets to skip a render. You've built the optimization and quietly disabled it.

**Option B — `useCallback`**

```javascript
const handleSelect = useCallback((id) => setSelected(id), []);
<ResultsList onSelect={handleSelect} />
```

**Why this wins:** `useCallback` returns the _same function reference_ across renders as long as its dependencies haven't changed. Now `React.memo` on `ResultsList` sees identical props and correctly skips re-rendering.

**The core problem `useCallback` solves:** functions are recreated fresh on every render, which breaks reference-equality checks (`React.memo`, dependency arrays) that other optimizations rely on.

---

## 7. useMemo — "Filtering 5,000 products shouldn't lag every time I toggle dark mode"

**Scenario:** A product page filters/sorts a large array based on a price range. The page also has an unrelated dark-mode toggle that triggers re-renders.

**Option A — recompute the filter inline in render**

```javascript
const filtered = products.filter(p => p.price <= maxPrice).sort(...);
```

Problem: This expensive filter+sort runs on _every_ re-render — including ones caused by totally unrelated state like the dark-mode toggle. With 5,000 items, that's a visible jank on every keystroke or click elsewhere on the page.

**Option B — `useMemo`**

```javascript
const filtered = useMemo(
  () => products.filter(p => p.price <= maxPrice).sort(...),
  [products, maxPrice]
);
```

**Why this wins:** React caches the result and only recomputes it when `products` or `maxPrice` actually change. Toggling dark mode no longer re-runs the filter.

**The core problem `useMemo` solves:** expensive calculations re-running on every render even when their actual inputs haven't changed.

---

## 8. useLayoutEffect — "A tooltip flickers/jumps position for a split second on open"

**Scenario:** A tooltip opens next to a button. You measure the button's position and the tooltip's own size (`getBoundingClientRect()`) to place it correctly — but users see it flash at the wrong position (e.g. top-left corner) for a frame before snapping to the right spot.

**Option A — `useEffect`**

```javascript
useEffect(() => {
  const rect = tooltipRef.current.getBoundingClientRect();
  setPosition(calculatePosition(rect)); // causes a second render
}, []);
```

Problem: `useEffect` runs _after_ the browser has already painted the screen. So the sequence is: render at wrong position → **paint (user sees it)** → effect runs → measure → re-render at correct position → paint again. That gap between the two paints is the visible flicker.

**Option B — `useLayoutEffect`**

```javascript
useLayoutEffect(() => {
  const rect = tooltipRef.current.getBoundingClientRect();
  setPosition(calculatePosition(rect));
}, []);
```

**Why this wins:** `useLayoutEffect` runs synchronously _after render but before the browser paints_. So React renders → measures/repositions in the layout effect → **then** paints once, already in the right spot. No visible jump.

**The core problem `useLayoutEffect` solves:** any time you need to measure the DOM (size, position, scroll) and synchronously adjust something _before the user sees a frame_ — the tiny "flash of wrong content" problem `useEffect` can't prevent.

**Trade-off / why it's not the default:** it's synchronous and blocks the browser from painting until it finishes, so overusing it can hurt perceived performance. Rule of thumb: reach for `useEffect` by default; switch to `useLayoutEffect` only when you have a visible flicker/measurement problem like above.

---

## 9. useImperativeHandle — "A parent needs to imperatively call `.focus()` or `.reset()` on a custom child component"

**Scenario:** You built a reusable `<CustomInput />` wrapping a styled `<input>`. A parent form wants to call `customInputRef.current.focus()` on validation failure — but `ref` on a custom component doesn't expose the DOM node by default.

**Option A — pass the raw DOM ref up and let the parent poke at it directly** Problem: leaks internal implementation detail (parent now assumes there's a literal `<input>` inside, breaks if you refactor internals) and exposes far more than the parent should be allowed to touch (parent could set `.value` directly, bypassing React state).

**Option B — `useImperativeHandle`**

```javascript
const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ''; }
  }));
  return <input ref={inputRef} {...props} />;
});
```

**Why this wins:** the parent gets a controlled, intentional API surface (`focus()`, `clear()`) instead of the raw DOM node — you decide exactly what's exposed.

**The core problem it solves:** letting a parent imperatively trigger behavior on a child component without exposing (and locking yourself into) the child's internal DOM structure.

---

## 10. useTransition — "Typing in a filter box feels laggy because it re-renders a huge list on every keystroke"

**Scenario:** Same big-list problem as `useMemo` earlier, but here the _list itself_ needs to change (not just be filtered from a cached computation) — e.g. switching between entirely different tabs/views of thousands of items, where even the render itself is heavy.

**Option A — just setState normally**

```javascript
setQuery(e.target.value); // input feels sluggish, typing lags behind
```

Problem: React treats this list re-render as urgent, same priority as the keystroke itself — so the browser can't keep the input feeling responsive while it's busy rendering thousands of list items.

**Option B — `useTransition`**

```javascript
const [isPending, startTransition] = useTransition();
setInputValue(e.target.value);       // urgent: keeps input responsive
startTransition(() => setQuery(e.target.value)); // low-priority: list can lag behind
```

**Why this wins:** it tells React "this specific update can be interrupted/deprioritized" — the input stays instantly responsive while the expensive list update happens in the background, with `isPending` available to show a subtle loading state.

**The core problem it solves:** letting you mark _some_ state updates as non-urgent so React keeps the UI responsive to input, instead of every update competing at the same priority.

---

## Hooks that exist but you'll rarely reach for directly

- **`useDeferredValue`** — similar goal to `useTransition` but for deferring a _value_ rather than wrapping a state setter (e.g. deferring the `query` used to filter a list, without controlling where it's set).
- **`useId`** — generates a stable unique ID for accessibility attributes (`aria-describedby`, label `htmlFor`) that's consistent between server and client rendering — avoids ID mismatches in SSR apps.
- **`useSyncExternalStore`** — the low-level hook that libraries like Redux/Zustand are built on, for subscribing to state that lives outside React. You'd write a custom hook using this maybe once if building your own state library; otherwise you use the library's hook instead.
- **`useDebugValue`** — labels a custom hook's value in React DevTools. Dev-tooling only, no runtime effect.

---

## Quick decision guide

| Problem you're facing                                                      | Reach for             |
| -------------------------------------------------------------------------- | --------------------- |
| "UI needs to change when data changes"                                     | `useState`            |
| "I need to sync with something outside React (fetch, subscription, timer)" | `useEffect`           |
| "I need the real DOM node, or a value that shouldn't trigger re-render"    | `useRef`              |
| "Too many components are just relaying a prop downward"                    | `useContext`          |
| "Several state fields must update together / consistently"                 | `useReducer`          |
| "A memoized child re-renders because I pass a new function each time"      | `useCallback`         |
| "An expensive calculation re-runs on unrelated re-renders"                 | `useMemo`             |
| "DOM measurement causes a visible flicker/jump before paint"               | `useLayoutEffect`     |
| "Parent needs to call a method on a custom child component"                | `useImperativeHandle` |
| "Typing/input feels laggy because of a heavy re-render"                    | `useTransition`       |

```
### useState
```

use for simple piece of data. Updates are batched so changes happen later not immediately.

```
const [count, setCount] = useState(0);

setCount(count + 1);  //BAD!!!!! Value of count might be stale
setCount(oldCount => oldCount + 1); // GOOD!!!  Always newest!
```

### useEffect

use for sideffects. No dependency means, run on every runder, empty dependency means run only once.

```
import { useState, useEffect } from 'react';

function DataFetcher() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Step 1: Fetch data
    fetch('https://api.example.com/users')
      .then(res => res.json())
      .then(users => setData(users));

    // Step 2: Cleanup (runs before next effect or unmount)
    return () => console.log('Cleanup: Cancel fetch if needed');
  }, []); // Empty list = run once on mount

  return <ul>{data?.map(user => <li key={user.id}>{user.name}</li>)}</ul>;
}


##INFINTE LOOPS##
function BadCounter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    setCount(count + 1) OR EVEN setCount(c => c + 1); // Runs every render → infinite!
  }); // NO deps array
  return <div>{count}</div>; // Freezes browser
}


##SELF DEPS##
useEffect(() => {
  setCount(count + 1);
}, [count]); // count changes → re-run → loop!

##FIX##

  useEffect(() => {
    const id = setInterval(() => setCount(c => c + 1), 1000);
    return () => clearInterval(id); // No leak/loop
  }, []); // Empty deps
  


```

### useRef

### useMemo

### useContext

### useReducer

### useCallback

### useLayoutEffect

### useTransition

### useDeferredValue
