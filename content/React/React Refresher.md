---
{"publish":true,"created":"2026-01-29T17:18:22.000Z","modified":"2026-08-17T03:03:56.514Z"}
---

###### SSR vs CSR

- SSR better for SEO
- FCP first contentful paint is quick.
- Perceived pageload time increase because user can see FCP quickly.
- Server receives request fetches necessary dataand then renders the html using **renderToString()** , **renderToPipeableStream()** and sends rendered html w/o javascript.
- javscript like evenhandlers and listeners are addded by the browser later on called **Hydration**. For this it uses hydratRoot() instead of createRoot().
- Hybrid approach can be used :
  - SEO frinedly pages like landing, blogs, product pages in SSR
  - Pages being login wall, dahsboards, admin panels, highly interactve apps like text editor
- Bundling wise we create seperate bundles :

```
const client = {
  entry: './src/client/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'public')
  }
  // Browser optimizations, code splitting, etc.
};

const server = {
  entry: './src/server/index.js',
  target: 'node', // Critical: tells webpack this runs in Node
  output: {
    filename: 'server.js',
    path: path.resolve(__dirname, 'build')
  },
  externals: [nodeExternals()] // Exclude node_modules from bundle
};

module.exports = [client, server];

```

###### Rendering Process

`Render Phase` : React calls component and produce JSX tree which says how the UI should look like based on current props and state. Its synchronous and pure (no side effects). ~~Can be interrupted (e.g., in Concurrent Mode).~~ Multiple renders may occur without committing to screen
`Reconciliation(Diff)` : React compares new jsx with old. Outputs the instructions for dom mutations without touching browser yet.
`Dom Commit` : React applies reconciliation plan by batching and executing actual browser DOM mutations. `useEffect` runs after dom commit.

###### Side Effects

- side effects means operations outside reacts core rendering process.
- Side effects include data fetching (api calls), subscriptions (websockets), timers, Manual DOM manipulation.
- `useEffect` is used to handle side effects.
- So For ex: An api call fetching data will only show case the data in the second dom commit. First dom commit shows loading state, since data is null, than useffect runs and fetched data than state updates cause rerender nd new data is shown from subsequent DOM commit.
- BAD TIMER EX :

```
function BadTimer() {
  const [count, setCount] = useState(0);
  setCount(1);
  count console
  
  // ❌ Side effects in render: Runs EVERY re-render!
  console.log('Rendering...');  // Logs multiple times
  setInterval(() => setCount(c => c + 1), 1000);  // Creates new interval every render!
  
  return <div>Count: {count}</div>;
}

```

- GOOD TIMER EX :
  ```function GoodTimer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // ✅ Runs once after mount, cleans up on unmount
    const id = setInterval(() => setCount(c => c + 1), 1000);
    return () => clearInterval(id);  // Prevents leaks
  }, []);  // Empty deps: runs once

  return <div>Count: {count}</div>;  // Pure render
  ```

}
