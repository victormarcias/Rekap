# Tree Shaking

The bundler removes exported code that nobody imports from the final bundle. A generic concept of any modern bundler (Webpack, Rollup, esbuild, Vite) — not specific to React.

## Why it works with ES Modules and not CommonJS

ES Modules' `import`/`export` are **static**: the bundler can analyze the code without running it and know exactly which exports of each module are used anywhere. CommonJS's `require()` is **dynamic** (can be called conditionally, with a string built at runtime) — the bundler can't guarantee ahead of time what will be needed, so it can't safely discard anything.

```js
// ✅ ES Modules: the bundler statically sees only `formatDate` is used
import { formatDate } from './utils';

// ❌ CommonJS: in theory the path could vary at runtime, the bundler can't analyze it the same way
const utils = require('./utils');
```

See the full example with `lodash` (importing the whole library vs. `lodash/debounce`) in [Frontend Diagnostics](../diagnostics/frontend.md#unoptimized-third-party-components).

## `sideEffects` in `package.json`

The bundler can't remove a module if running it has side effects (e.g. it modifies a global object, registers something) even if nothing imports its exports — just in case, it keeps it. A library can explicitly declare that its files have **no** side effects, so the bundler can confidently discard them if unused.

```json
{
  "name": "my-library",
  "sideEffects": false
}
```

```json
// or a specific list of files that do have them (e.g. a self-executing polyfill)
{
  "sideEffects": ["./src/polyfills.js"]
}
```

Without this declaration, the bundler assumes **everything** might have side effects and is more conservative — it discards less code than it actually could.

---
Related: [Frontend Diagnostics](../diagnostics/frontend.md), [Performance Diagnostics](performance-diagnostics.md) (bundle analyzer, to see what's left inside).
