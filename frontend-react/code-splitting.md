# Code Splitting / Lazy Loading

Splitting the JS bundle into pieces that load **when needed**, instead of shipping the entire app in a single file on the first request. A user landing on the login page doesn't need to download the admin panel's code yet.

## `React.lazy` + `Suspense`

```jsx
import { lazy, Suspense } from 'react';

// the dynamic import() tells the bundler (Webpack/Vite) this component
// goes in a separate chunk, not in the main bundle
const AdminPanel = lazy(() => import('./AdminPanel'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      {/* the AdminPanel chunk is only requested over the network the first time
          this component tries to render */}
      <AdminPanel />
    </Suspense>
  );
}
```

`Suspense` shows the `fallback` while the chunk hasn't arrived yet — it's the same mechanism SSR/streaming frameworks use to avoid blocking the full render while waiting on one slow part.

## Where it's typically applied

- **Routes**: each page of a router loads its own chunk — the most common and highest-impact case (nobody needs the checkout's JS while looking at the home page).
- **Heavy conditional components**: a complex modal, a rich text editor, a chart using a big library — things not visible on the first render.

```jsx
// split by route with React Router
const Home = lazy(() => import('./pages/Home'));
const Checkout = lazy(() => import('./pages/Checkout'));

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/checkout" element={<Checkout />} /> {/* its JS isn't even requested until you land here */}
</Routes>
```

## Not to be confused with Tree Shaking or Module Federation

All three reduce how much JS ends up running in the browser, but at different moments:

- **[Tree shaking](tree-shaking.md)**: at **build time**, eliminates code that's never used anywhere — it never even ends up in any bundle.
- **Code splitting**: the code is used, but its download is **deferred** until the moment it's needed — it's still part of the app, just in a different file.
- **[Module Federation](module-federation.md)**: splits the app into pieces that don't even share the same build/deploy — code splitting splits a single app's bundle; Module Federation composes bundles from independent apps at runtime.
