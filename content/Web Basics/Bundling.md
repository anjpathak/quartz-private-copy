---
{"publish":true,"created":"2026-01-28T12:10:04.000Z","modified":"2026-08-17T03:04:11.270Z"}
---

- Bundlers available Vite vs Webpack vs Rollup vs ES6Modules are native way
- Vite doesn't tree shake in dev mode. For production it uses rollup under the hood which has excellent tree shaking capabilities
- Rollup is particularly good at producing cleaner output with better tree shaking than Webpack
- Webpack requires config for tree shaking in prod.
- Vite does it automatically w/o needing extra config
- both vite and webpack doesn't work for common js module syntax like
  `// ✅ Works with tree shaking
  `import { debounce } from 'lodash-es';\`

`// ❌ Doesn't work with tree shaking`\
`const { debounce } = require('lodash');`

- Always use specific imports :
  `// ❌ Bad - imports entire lodash (~70KB)
  `import \_ from 'lodash';
  \`\_.debounce(fn, 300);

`// ✅ Good - only imports debounce (~2KB)
`import { debounce } from 'lodash-es';
`debounce(fn, 300);
`
**sideEffects are major concern.**
Some files like .css .scss etc have sideeffects that means they execute some code import css or set global object values like in Polyfills.js just by importing tham even if the imports are not used anywhere.
We dont want to treeshake such files. So we add either :

```
// package.json
{
  "sideEffects": ["**/*.css"]  // Says: CSS has side effects
}

```

or

```
// vite.config.js
export default {
  build: {
    rollupOptions: {
      treeshake: {
        moduleSideEffects: false  // Says: NOTHING has side effects
      }
    }
  }
}
```

when building a library or npm package we tell our consumers aboout files having sideeffects using package.json

#### **VITE** VS **WEBPACK** VS **ROLLUP**

```
WEBPACK
```

- Bundle Based, bundle app before serving hence slow dev server and HMR
- great plugin and loader ecosystem
- ==Solves complex build requirements like Module Federation which vite lacks==
- Complex Custom Loaders tha vite misses
- Better for handling commonjs modules, handles old npm packages better.
- Good for a large scale

  **Vite**
- Native ESM-based, Serves source files as native ES modules (no bundling!)
- Instant server startup - no matter how large your app (< 1 second)[](https://www.syncfusion.com/blogs/post/webpack-vs-vite-bundler-comparison)​
- Lightning-fast HMR - updates in milliseconds
- Only works well with modern browsers in development
- Different behavior between dev and prod (dev uses native ESM, prod uses Rollup)
- \==Vite is basically ES Modules + JSX/TypeScript transformation + HMR + 1. Dependency pre-bundling (optimize node\_modules) + CSS modules

  **Roll Up**
- Module bundler optimized for libraries
- Produces clean, readable output, readable by user better for debugging. Since use case is for libraries, readable final code is acceptable and useful.
- Best Tree shaking
- Not ideal for applications (lacks dev server, HMR out-of-box)
- Doesn't bundle dependencies, since library is not a standalone app and so requires no dependencies. Dependencies are eventually bundle by the consuemr together with your library. This keeps the library package size smaller. Nobody likes a 200kb library
- Multiple output formats (apps only need one format)

```// rollup.config.js
export default {
  input: 'src/index.js',
  output: [
    { file: 'dist/my-lib.cjs.js', format: 'cjs' },     // CommonJS (Node)
    { file: 'dist/my-lib.esm.js', format: 'es' },      // ESM (modern)
    { file: 'dist/my-lib.umd.js', format: 'umd' }      // UMD (browsers)
  ]
}
```

Your library might be used in:
Node.js apps (need CommonJS)\
Modern bundlers (need ESM)\
Direct browser scripts (need UMD)\
**Apps only need ONE bundle format.**

## **Code Splitting**

```
Route Based : works great when routes have distinct functionaly and boundaries. Less shared code between routes.
```

```
const Home = lazy(() => import('./pages/Home'))
  Webpack default config chunk: async makes Home a seperate chunk.
```

```
  Component Based splitting like lazy loaded heavy components VideoPlayer/Charts/Rich Text Editors/ Heavy Modal
```

```
const HeavyModal = lazy(() => import('./components/HeavyModal')); const VideoPlayer = lazy(() => import('./components/VideoPlayer'));
```

```
Vendor/Library Splitting == these chunks dont change much so can be cached for second time use.
```

```
// vite.config.js
export default {
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // React vendor chunk
          'vendor-react': ['react', 'react-dom', 'react-router-dom'],
          
          // UI library chunk
          'vendor-ui': ['@mui/material', '@emotion/react', '@emotion/styled'],
          
          // Utilities chunk
          'vendor-utils': ['lodash-es', 'axios', 'date-fns'],
        }
      }
    }
  }
}

```

```
Dynamic Imports : 
```

```const { Chart } = await import('chart.js');
similar to lazy loading but this is js syntax and React's Lazy() is used specifically for react components.
```

==React.lazy() can be used for loading third-party libraries, but with an important constraint: it only works with default exports==

```
async function loadLocale(locale) {
  switch (locale) {
    case 'en':
      return import('./locales/en.json');
    case 'es':
      return import('./locales/es.json');
    case 'fr':
      return import('./locales/fr.json');
    case 'hi':
      return import('./locales/hi.json');
  }
}

// User selects language
const translations = await loadLocale(userLanguage);

```

```
VITE SPECIFIC MAGIC COMMENTS
```

```
const Dashboard = lazy(() => import(
  /* webpackChunkName: "dashboard" */     // Name the chunk
  /* webpackPrefetch: true */             // Prefetch during idle time
  './pages/Dashboard'
));

const AdminPanel = lazy(() => import(
  /* webpackChunkName: "admin" */
  /* webpackPreload: true */              // Preload in parallel with parent
  './pages/AdminPanel'
));
```

\==These are Webpack comments but Vite/Rollup respects webpackChunkName
PREFETCH is low priority, fetcched on idle time
PRELOAD is medium priority fetched parallely with parent chunk

**Real World WEBPACK Example**

```
// webpack.config.js for large app
module.exports = {
  entry: {
    main: './src/index.js',
  },
  
  optimization: {
    splitChunks: {
      chunks: 'all',
      maxInitialRequests: 25,
      minSize: 20000,
      cacheGroups: {
        // Critical vendors (React, etc.)
        criticalVendors: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
          name: 'critical-vendors',
          priority: 20,
        },
        
        // Heavy UI libraries
        uiVendors: {
          test: /[\\/]node_modules[\\/](@mui|@material-ui|@emotion)[\\/]/,
          name: 'ui-vendors',
          priority: 15,
        },
        
        // Analytics/monitoring
        monitoring: {
          test: /[\\/]node_modules[\\/](datadog|newrelic|sentry)[\\/]/,
          name: 'monitoring',
          priority: 12,
        },
        
        // Everything else from node_modules
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
        },
        
        // Shared application code
        common: {
          minChunks: 2,
          name: 'common',
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },
    
    // Runtime chunk for Webpack runtime code
    runtimeChunk: {
      name: 'runtime',
    },
  },
  
  output: {
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
  },
};

```
