# DOM Events

How events propagate in the browser — a web platform concept, not specific to any framework.

## Bubbling vs Capturing

An event (e.g. a click) travels through the DOM in two phases: first **capturing** (goes down from `window` to the element where the click happened), then **bubbling** (goes back up from that element to `window`). By default, `addEventListener` listens on the bubbling phase.

```js
parent.addEventListener('click', () => console.log('parent'));
child.addEventListener('click', () => console.log('child'));

// a click on child prints: "child" then "parent" (bubbling, default)

parent.addEventListener('click', () => console.log('parent capturing'), { capture: true });
// with capture: true, this listener runs BEFORE the event reaches the target
```

## Event delegation

Instead of putting a listener on every child element, you put **just one on the parent** and use bubbling to figure out which child the click happened on (`event.target`). Fewer listeners = less memory, and it works automatically with children added later (no need to re-attach anything).

```js
// ❌ one listener per <li>, and you have to add a new one every time an <li> is created
document.querySelectorAll('li').forEach(li => li.addEventListener('click', handleClick));

// ✅ a single listener on the <ul>, works for present and future children
document.querySelector('ul').addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') handleClick(e);
});
```

## `preventDefault` vs `stopPropagation`

Often confused because both "stop" something, but different things:

- **`preventDefault()`**: cancels the browser's default behavior for that event (e.g. an `<a>` navigating, a form submitting). Doesn't affect propagation — the event keeps bubbling normally.
- **`stopPropagation()`**: cuts off the event's propagation to the next elements in the chain (stops bubbling/capturing further). Doesn't affect the browser's default behavior.

```js
form.addEventListener('submit', (e) => {
  e.preventDefault();      // ✅ avoids the native submit's page reload
  // e.stopPropagation();  // this is NOT needed for the above — they're independent things
});
```

## Note: `SyntheticEvent` in React

React doesn't attach a native listener to every element — it attaches **a single listener** at the root of the tree (event delegation at the whole-app scale) and wraps the native event in a `SyntheticEvent` with a consistent cross-browser API. The logic above (bubbling, delegation, `preventDefault`) still applies the same way, React just already implements it for you at the framework level.

---
Related: [Frontend Diagnostics](../diagnostics/frontend.md) (incorrect event handling, debounce/throttle).
