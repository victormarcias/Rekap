# React — Fundamentals

How React works internally, before getting into hooks (see [Hooks](hooks.md)) or performance (see [Frontend Diagnostics](../diagnostics/frontend.md)).

## Virtual DOM

A **Virtual DOM** is a lightweight tree of JS objects representing the desired UI — it doesn't touch the real DOM. When state changes, React builds a new Virtual DOM, compares it (*diffing*) against the previous one, and only applies the minimal necessary changes to the real DOM (*reconciliation*).

```jsx
// changing just one <li>'s text doesn't recreate the entire list —
// React compares the previous tree against the new one and updates
// only the text node that changed
function List({ items }) {
  return <ul>{items.map(item => <li key={item.id}>{item.text}</li>)}</ul>;
}
```

**Why it exists**: manipulating the real DOM is expensive (every change can trigger a browser layout/paint). Comparing JS objects in memory is cheap. The Virtual DOM lets React figure out *what* changed without touching the DOM at every intermediate step, and apply all the real changes at once, in the minimum number of operations possible.

**The role of `key`**: when React reconciles a list, it uses `key` to identify which element is which between one render and the next — without a stable `key`, React can confuse "an item got reordered" with "one got deleted and a new one got created," unnecessarily losing those components' internal state (see [Missing `key` in lists](../diagnostics/frontend.md#missing-key-in-lists)).

## JSX

Syntax that mixes HTML-like markup with JS, but **isn't HTML** — it's syntactic sugar that a compiler (Babel/SWC) transforms into function calls before the browser sees a single line. It's **React**'s way (web and React Native alike) of describing the Virtual DOM tree from above — it's not exclusive to web, it's exclusive to React as a library.

```jsx
// this...
const element = <h1 className="title">Hello {name}</h1>;

// ...transpiles to this (React 17+, "new" JSX transform):
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', { className: 'title', children: `Hello ${name}` });
```

That's why JSX can use `{}` to insert any valid JS expression (variables, functions, ternaries) — at compile time it just ends up as one more argument to a normal function call. And that's why a React component **has** to return valid JSX (or `null`) — it's not free-form HTML, it follows the rules of a JS expression (e.g. `class` doesn't exist, it's `className`, because `class` is a reserved word in JS).

## The same pattern on Mobile

"Declaratively describe how the UI looks, diff against the previous version, apply only the minimal real change" isn't an idea exclusive to React — it's the convergent solution of almost every modern declarative UI framework. Each ecosystem arrived at this separately because the underlying problem is the same: recalculating the *entire* real UI tree on every state change is extremely expensive, regardless of whether that real UI is the browser's DOM or a native view.

### Swift (SwiftUI, iOS)

`View`s are immutable structs describing the desired UI; SwiftUI compares them against the previous tree and updates only what changed in the real render.

```swift
struct CounterView: View {
    @State private var count = 0
    var body: some View {
        Button("Count: \(count)") { count += 1 }
    }
}
// when `count` changes, SwiftUI recalculates `body`, compares against the previous
// tree, and updates only the button's text — same mechanism as React
```

### Kotlin (Jetpack Compose, Android)

Same mechanism — `@Composable` functions describe the UI, Compose does its own diffing ("recomposition") and updates only the affected parts.

```kotlin
@Composable
fun CounterView() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) { Text("Count: $count") }
}
// when `count` changes, Compose "recomposes" only what's affected — same mechanism, different name
```

### React Native

Literally the same React/Virtual DOM as the web version, but with a different renderer at the end — instead of applying changes to browser DOM nodes, it applies them to real native views (`UIView` on iOS, `View` on Android). It's **React DOM** (the web renderer) that's web-specific, not React or the Virtual DOM itself — React Native reuses exactly the same core and the same JSX from above. For the practical side (components, styling, navigation, the bridge to native code) see [React Native — Fundamentals](../stacks/react-native/fundamentals.md).

---
Related: [Hooks](hooks.md), [Frontend Diagnostics](../diagnostics/frontend.md).
