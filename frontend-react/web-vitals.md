# Web Vitals

Concrete metrics Google defines to measure "how good it feels" to load and use a page — replacing vague intuitions ("feels slow") with measurable numbers, and they're a search ranking signal. Measured by [Lighthouse](performance-diagnostics.md#lighthouse-generic), among other tools.

## Core Web Vitals

- **LCP (Largest Contentful Paint)**: how long it takes for the largest visible element on the initial screen to paint (a hero image, a large text block). It's the "does it feel loaded yet?" metric — a good LCP is under 2.5s. Typical causes of a bad LCP: unoptimized images, render-blocking fonts, JS that delays when content appears (see [CSR vs SSR](rendering.md#csr-vs-ssr)).
- **INP (Interaction to Next Paint)**: how long the UI takes to respond visually after the user interacts (click, tap, keystroke) — replaced FID (First Input Delay) because it measures the **complete** interaction, not just the first input. A high INP usually comes from JS blocking the main thread at the moment of interaction (see [Main thread blocking](../diagnostics/frontend.md#main-thread-blocking)).
- **CLS (Cumulative Layout Shift)**: how much the layout "jumps" while the page loads — images or ads with no reserved `width`/`height` that push content below when they finish loading, a banner that appears late and shoves everything down. Measured by summing how much each visible element moved, weighted by how big the jump was.

```html
<!-- ❌ no reserved dimensions: when the image loads, it pushes the text below (high CLS) -->
<img src="banner.jpg" />

<!-- ✅ the browser reserves the space from the first render, even if the image takes a while to arrive -->
<img src="banner.jpg" width="800" height="400" />
```

## Why they matter more than "the page loads fast" in general

A low total load time doesn't guarantee a good experience if the main content takes a while to appear (bad LCP), if the page feels stuck when touched (bad INP), or if the user ends up clicking the wrong spot because something moved (bad CLS). The three measure different moments of the experience — initial load, responsiveness, and visual stability — which is why no single one is enough to say "this page performs well."

---
Related: [Performance Diagnostics](performance-diagnostics.md), [Frontend Diagnostics](../diagnostics/frontend.md).
