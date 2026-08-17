---
{"publish":true,"created":"2025-12-23T09:48:58.000Z","modified":"2026-08-17T03:03:59.103Z"}
---

**esbuild** excels at **speed** for dev (10-100x faster) but lacks **advanced optimizations** needed for production. **Rollup/Rolldown** prioritizes **tree-shaking + bundle quality** over raw speed, perfect for prod but too slow for dev HMR loops.[](https://vite.dev/guide/build)​

## esbuild: Dev Speed King

|Strength|Why Dev Only|
|---|---|
|**Blazing fast** (Go + parallel)|Instant server start + HMR on large apps [](https://github.com/vitejs/vite/discussions/7622)​|
|**Pre-bundles deps** once|`node_modules` → ESM instantly [](https://vite.dev/guide/build)​|
|**On-demand transforms**|JSX/TS/CSS → browser ES modules [](https://vite.dev/guide/build)​|

**Weak for prod**:

- Poor **tree-shaking** (dead code elimination)[](https://github.com/vitejs/vite/discussions/7622)​

- Limited **code-splitting** + format options[](https://vite.dev/guide/build)​

- No advanced **plugin ecosystem**[](https://github.com/vitejs/vite/discussions/7622)​

## Rollup/Rolldown: Prod Optimization King

|Strength|Why Prod Only|
|---|---|
|**Perfect tree-shaking**|Removes 100% unused code [](https://vite.dev/guide/build)​|
|**Smart code-splitting**|Dynamic imports → optimal chunks [](https://vite.dev/guide/build)​|
|**Multiple formats**|ESM/CJS/UMD/IIFE [](https://vite.dev/guide/build)​|
|**Rich plugins**|Terser minification, etc. [](https://vite.dev/guide/build)​|

**Weak for dev**:

text

`npm run dev → 5-30s startup + 1-3s HMR 😴 vs esbuild → 100ms startup + 20ms HMR ⚡`

## Vite's Genius Hybrid

text

`Dev:  esbuild pre-bundle deps → native ESM dev server      ↓ (npm run build) Prod: Rollup/Rolldown → optimized dist/ bundle`

**Rolldown** (Rust Rollup) = **faster Rollup** (builds 5-10x quicker) but **still Rollup-quality** tree-shaking. Dev stays esbuild.[](https://www.vuemastery.com/blog/the-build-process-of-a-vue-app-rollup-vs-rolldown/)​

**Result**: Best dev UX + best prod bundle = Vite wins.[](https://vite.dev/guide/build)​
