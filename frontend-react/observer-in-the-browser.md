# Observer Pattern in the browser

The Observer pattern in general (an object notifies a list of dependents when it changes) is already covered in [Behavioral Patterns](../system-design/behavioral-patterns.md#observer). Here's the frontend-specific angle: the browser ships the pattern already solved as native APIs, replacing code that used to be solved with manual polling (and its performance cost).

## `MutationObserver`

Notifies when the DOM changes (nodes added/removed, an attribute changes), without having to check the DOM by hand on an interval.

```js
const observer = new MutationObserver((mutations) => {
  mutations.forEach(m => console.log('Changed:', m.type));
});

observer.observe(document.getElementById('list'), { childList: true, attributes: true });
// ✅ finds out about every change as soon as it happens — no setInterval comparing the DOM every X ms
```

## `IntersectionObserver`

Notifies when an element enters or leaves the viewport (or another container element). Replaces the old pattern of listening to the `scroll` event and manually calculating positions — much more expensive because `scroll` fires dozens of times per second.

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) loadImage(entry.target); // real lazy loading
  });
});

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

**Typical use case**: image lazy loading, infinite scroll, animations that start when entering the viewport — all without a single `scroll` listener. See also [Debounce / Throttle](../diagnostics/frontend.md#incorrect-event-handling), the alternative when you do need to listen to `scroll` directly.

## `ResizeObserver`

Notifies when an element's size changes — not the whole window (`window.resize`), but a specific element, even if it changed via CSS (a flex/grid that repositioned) with the window never being touched.

```js
const observer = new ResizeObserver((entries) => {
  entries.forEach(entry => console.log('New size:', entry.contentRect.width));
});

observer.observe(document.querySelector('.resizable-panel'));
```

## The common pattern

All three replace the same bad alternative: checking something in a loop (`setInterval`) or reacting to an overly noisy event (`scroll`, window `resize`) to detect a specific change. The native API notifies only when the specific change happens — it's Observer applied to the web platform instead of a custom class.
