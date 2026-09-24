# Error Boundaries

An error boundary is a component that **catches JS errors thrown during the render** of its child components, and shows a fallback UI instead of the whole app breaking to a blank screen. Without one, an error in any component takes down the entire React tree containing it.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true }; // triggers the fallback render on the next render
  }

  componentDidCatch(error, info) {
    logErrorToService(error, info); // side effect: log, report to Sentry, etc.
  }

  render() {
    if (this.state.hasError) return <h2>Something went wrong.</h2>;
    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary>
      <Dashboard /> {/* if Dashboard throws a render error, only this part gets replaced */}
    </ErrorBoundary>
  );
}
```

## Why it's a class (there's no hook equivalent)

`getDerivedStateFromError` and `componentDidCatch` still don't have a hooks equivalent — it's one of the few legitimate reasons to still write a class component in modern React code. In practice almost nobody writes it by hand: the `react-error-boundary` library is used instead, exposing the same functionality with a reusable component API.

```jsx
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary fallback={<h2>Something went wrong.</h2>} onError={logErrorToService}>
  <Dashboard />
</ErrorBoundary>
```

## What it does NOT catch

An error boundary only catches errors during **render**, in lifecycle methods, and in child components' constructors. It doesn't catch errors in: event handlers (`onClick`), async code (`setTimeout`, promises), server-side rendering, or errors thrown in the error boundary itself. Errors from an `onClick` are handled with a normal `try/catch` inside the handler.

## Where to place them

You don't need a single global error boundary — it's common to put one around independent UI sections (e.g. a widget that consumes an unreliable external API), so an error there doesn't take down the rest of the page that's still working fine.

---
Related: [Hooks](hooks.md), [Frontend Diagnostics](../diagnostics/frontend.md).
