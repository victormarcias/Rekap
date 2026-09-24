# Global State: Context API vs Redux

## Prop drilling — the problem

Passing a prop through several intermediate components that don't use it, just so it reaches a child component that does need it. The deeper the tree, the more intermediate components end up coupled to a prop that's irrelevant to them — and changing that data's shape forces you to touch every intermediate one.

```jsx
// user travels through Layout and Sidebar without either of them using it
function App({ user }) {
  return <Layout user={user} />;
}
function Layout({ user }) {
  return <Sidebar user={user} />;
}
function Sidebar({ user }) {
  return <UserBadge user={user} />; // the only one that actually needs it
}
```

## Context API — the native solution

Provides a value at one point in the tree and makes it directly available to any descendant via `useContext`, without going through the intermediate components (see [`useContext`](hooks.md#usecontext)).

```jsx
const UserContext = createContext(null);

function App({ user }) {
  return (
    <UserContext.Provider value={user}>
      <Layout /> {/* no longer needs to receive or forward user */}
    </UserContext.Provider>
  );
}
function UserBadge() {
  const user = useContext(UserContext); // reads directly, regardless of depth
  return <span>{user.name}</span>;
}
```

**Where it falls short**: Context isn't an optimized state manager — any component consuming that Context re-renders every time the `value` changes, even if the component only uses part of that value. A Context with a large object that changes often (e.g. an entire shopping cart's state updated on every click) can over-render components that didn't care about that specific change.

## Redux (or other external libraries)

When global state is large, changes often, or needs complex update logic (several actions modifying it from different places), a dedicated library (Redux, Zustand, Jotai) adds what Context lacks: selectors that subscribe a component only to the slice of state it cares about (avoiding extra re-renders), DevTools for inspecting every state change step by step, and an explicit pattern for how state updates (actions + reducers in Redux).

```jsx
// Redux Toolkit — minimal example
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addItem: (state, action) => { state.items.push(action.payload); },
  },
});

function CartBadge() {
  // only re-renders if items.length changes — not if
  // any other part of the app's global state changes
  const count = useSelector(state => state.cart.items.length);
  return <span>{count}</span>;
}
```

## When to use each one

- **Plain prop drilling**: if it's 1-2 levels, sometimes it's simpler than adding Context — not every prop chain deserves an abstraction.
- **Context**: state that changes rarely (theme, language, logged-in user) with no high-frequency updates.
- **Redux/Zustand**: state that changes often, with multiple update sources, or where re-render performance is already a measured problem (not an assumption — see [Performance Diagnostics](performance-diagnostics.md)).

---
Related: [Hooks](hooks.md), [Frontend Diagnostics](../diagnostics/frontend.md).
