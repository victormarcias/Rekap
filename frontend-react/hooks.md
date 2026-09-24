# Hooks

Hooks are a React-specific API (and frameworks that copied the model, like Preact).

## Rules of hooks

1. **Only call hooks at the top level** of a component — never inside an `if`, a loop, or a nested function.
2. **Only call hooks from React components or from other hooks** — never from a plain JS function.

```jsx
// ❌ breaks rule 1: if the condition changes between renders, the hook order changes
function Component({ show }) {
  if (show) {
    const [value, setValue] = useState(0); // sometimes called, sometimes not
  }
}

// ✅ the hook is always called, the condition goes inside
function Component({ show }) {
  const [value, setValue] = useState(0);
  if (show) { /* use value here */ }
}
```

**Why this rule exists**: React doesn't identify each hook by name, but by the **order** in which they're called during render — internally it stores them in a list and associates them by position. If a hook is sometimes called and sometimes not, the order gets misaligned between renders and React assigns one hook the state that belonged to another.

## `useEffect`

Runs after the render, to sync the component with something external (fetch, subscription, timer, directly manipulating the DOM) — not for logic that can be resolved during the render itself.

```jsx
useEffect(() => {
  fetchData();
});             // no array: runs after EVERY render

useEffect(() => {
  fetchData();
}, []);         // empty array: runs once, on mount

useEffect(() => {
  fetchData();
}, [userId]);   // runs on mount, and again every time userId changes
```

**The dependency array isn't optional in practice**: without it (or with poorly declared dependencies), the effect fires more than it should — and if it sets state inside, each firing can force another render. See [Misused `useEffect`](../diagnostics/frontend.md#misused-useeffect--cascading-re-renders) for the concrete performance case.

## `memo`, `useMemo`, `useCallback` — memoization

```jsx
const total = useMemo(() => calculateHeavyTotal(items), [items]);   // memoizes a computed VALUE
const handleClick = useCallback(() => doSomething(id), [id]);        // memoizes a FUNCTION
const Row = memo(function Row({ item }) { ... });                     // memoizes an entire COMPONENT
```

- **`useMemo`**: avoids recomputing something expensive on every render, if its dependencies haven't changed.
- **`useCallback`**: avoids creating a new function on every render — matters when that function is a prop of a component wrapped in `memo`, because a new function breaks the reference comparison and defeats the memo.
- **`memo`**: wraps a component so it doesn't re-render if its props haven't changed (shallow comparison).

Don't memoize everything by default — it adds comparison overhead; use it where the avoided cost (an expensive render, or breaking a child's memo) justifies it. See [Components not using `memo`/`useMemo`/`useCallback`](../diagnostics/frontend.md#components-not-using-memousememousecallback) for the performance symptom it fixes.

## Custom hooks

A function that starts with `use` and calls other hooks inside — the way to extract reusable stateful logic between components, without duplicating code or resorting to heavier patterns (HOCs, render props).

```jsx
// ✅ custom hook: encapsulates fetch + loading + error, reusable in any component
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url).then(r => r.json()).then(setData).finally(() => setLoading(false));
  }, [url]);

  return { data, loading };
}

// usage: any component asks for only what it needs, without repeating the fetch logic
function UserProfile({ userId }) {
  const { data, loading } = useFetch(`/api/users/${userId}`);
  if (loading) return <Spinner />;
  return <div>{data.name}</div>;
}
```

A custom hook doesn't share state between the components using it — each call has its own instance of `useState`/`useEffect`, as if the code were copy-pasted (without actually being copied).

This example `useFetch` only covers `data`/`loading` — it's missing `error` and `retry` to be complete, see [Request States](request-states.md).

## `useRef`

Holds a mutable value that **persists between renders without causing a re-render** when it changes — unlike `useState`, writing to `ref.current` doesn't tell React anything changed. Two typical uses:

```jsx
// 1. Reference to a real DOM node (e.g. to focus it manually)
function SearchInput() {
  const inputRef = useRef(null);
  useEffect(() => { inputRef.current.focus(); }, []);
  return <input ref={inputRef} />;
}

// 2. Storing a value that needs to survive renders but shouldn't trigger a re-render
function Timer() {
  const intervalId = useRef(null);
  const start = () => { intervalId.current = setInterval(() => {}, 1000); };
  const stop = () => clearInterval(intervalId.current);
  return <button onClick={start}>Start</button>;
}
```

If the value needs to be reflected in the UI, it's `useState`; if it's internal "bookkeeping" the UI doesn't need to show, it's `useRef`.

## `useContext`

Reads a value provided higher up in the tree by a `Context.Provider`, without having to manually pass it prop by prop through every intermediate component (see [prop drilling](global-state.md#prop-drilling--the-problem)) — it's React's version of **Dependency Injection**: the `Provider` is the container defining what value is available, and `useContext` is asking for that injected dependency, without the component knowing where it came from.

```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  // Toolbar doesn't use the theme, but before it had to receive it anyway
  // to be able to pass it down to ThemedButton — with Context it no longer does
  return <ThemedButton />;
}

function ThemedButton() {
  const theme = useContext(ThemeContext); // 'dark' — reads directly, without going through Toolbar
  return <button className={theme}>Click</button>;
}
```

Any component using `useContext` re-renders when the Provider's `value` changes, no matter how deep it is in the tree — see [Global State: Context API vs Redux](global-state.md) for when this becomes a performance problem and what the alternatives are.

🐣 **Fun fact — where the name comes from**: "hooking into" React's internal features (state, lifecycle, context) from a plain function. Before hooks, only a class component had that (`this.state`, `componentDidMount`) — `useState` hooks into the state system, `useEffect` into the lifecycle, `useContext` into the Context tree. Same concept as a git hook: a hook point for inserting your own code into the behavior of a system that already exists.

---
Related: [Frontend Diagnostics](../diagnostics/frontend.md) (`useEffect`, `memo`/`useMemo`/`useCallback`), [Global State](global-state.md), [React Fundamentals](react-fundamentals.md).
