# Performance Diagnostics

Tools for measuring instead of guessing where a frontend app's performance problem is.

## Lighthouse (generic)

An automated audit that runs on any page (Chrome DevTools, CLI, or CI) and returns a Performance, Accessibility, Best Practices, and SEO score, with specific recommendations. Measures the [Core Web Vitals](web-vitals.md) (LCP, INP, CLS).

```bash
npx lighthouse https://myapp.com --view
```

## Bundle analyzer (generic)

Visualizes what's actually inside the final JS bundle — a treemap where each box is a module, its size proportional to the space it takes up. Useful for finding the 200kb dependency imported for a single function (see the full `lodash` vs `lodash/debounce` example in [Frontend Diagnostics](../diagnostics/frontend.md)).

```bash
# E.g. with Webpack
npx webpack-bundle-analyzer stats.json

# E.g. with Vite/Rollup
npx vite-bundle-visualizer
```

## React Profiler (React-specific)

A React DevTools tab that records a render session and shows, per component, how long each one took and **why** it re-rendered (which prop or state changed). It's the specific tool for answering "why is this component re-rendering so much?" instead of guessing by reading the code.

The other two (Lighthouse, bundle analyzer) tell you **how big/slow the initial load is**; the Profiler tells you **what's happening during interaction**, with the app already running.

---
Related: [Web Vitals](web-vitals.md), [Frontend Diagnostics](../diagnostics/frontend.md), [Tree shaking](tree-shaking.md).
