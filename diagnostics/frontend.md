# Frontend Diagnostics

Most common causes of client-side (React) slowness, from most to least frequent.

## Misused `useEffect` → cascading re-renders

Poorly declared (or missing) dependencies cause the effect to fire more than it should, and each firing can set state and force another render.

```jsx
// ❌ effect with no dependency array: runs on every render
useEffect(() => { fetchData(); });

// ✅ only runs when userId changes
useEffect(() => { fetchData(); }, [userId]);
```

Check with the **React DevTools Profiler**: how many times a component renders and why (the "why did this render" highlight).

## Missing `key` in lists

Without a stable `key` (or using the array index as key when the list gets reordered/filtered), React can't diff efficiently and re-renders/remounts more than necessary.

```jsx
// ❌ key = index, breaks if the list gets reordered
items.map((item, i) => <Item key={i} {...item} />)

// ✅ key = stable id from the data
items.map((item) => <Item key={item.id} {...item} />)
```

## Missing pagination / virtualization

Rendering thousands of DOM nodes at once saturates layout/paint. Paginate, or use *windowing* (`react-window`, `react-virtualized`) for long lists — only the items visible in the viewport get mounted.

```jsx
import { FixedSizeList } from 'react-window';

// ✅ only mounts the visible rows, not all 10,000 in the array
<FixedSizeList height={600} itemCount={items.length} itemSize={40}>
  {({ index, style }) => <div style={style}>{items[index].name}</div>}
</FixedSizeList>
```

## Components not using `memo`/`useMemo`/`useCallback`

Without memoization, a child component re-renders even when its props haven't changed (if the parent re-renders), or an expensive calculation gets repeated on every render.

```jsx
const total = useMemo(() => calculateHeavyTotal(items), [items]);
const handleClick = useCallback(() => doSomething(id), [id]);
const Row = memo(function Row({ item }) { ... });
```

Don't overdo it: memoizing everything adds comparison overhead — use it where the avoided cost (an expensive render, a prop that breaks a reference) justifies it.

## Incorrect event handling

Handlers created inline on every render (`onClick={() => f(id)}`) generate a new function each time, invalidating the memoization of children that depend on that prop. Missing *debounce/throttle* on high-frequency events (`scroll`, `resize`, search `input`) triggers excessive work.

```ts
// ✅ debounce with TypeScript, avoids 1 request per keystroke
function debounce<T extends (...args: any[]) => void>(fn: T, ms: number) {
  let timer: ReturnType<typeof setTimeout>;
  return (...args: Parameters<T>) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), ms);
  };
}
const onSearch = debounce((q: string) => fetchResults(q), 300);
```

## Complex calculations in render / excessive re-renders

Any heavy computation (`sort`, `filter`, transformations) run directly in the component body repeats on every render. Pull it out with `useMemo` or move it outside the component if it doesn't depend on props/state.

```jsx
// ❌ sorts on every render, even if items hasn't changed
const sorted = items.sort((a, b) => a.price - b.price);

// ✅ only recalculated when items changes
const sorted = useMemo(() => [...items].sort((a, b) => a.price - b.price), [items]);
```

## Heavy textures / images

Unoptimized images (real size, format) block paint and eat bandwidth. Use modern formats (WebP/AVIF), `srcset`/`loading="lazy"`, and an image CDN with on-the-fly resizing.

```jsx
// ✅ lazy loading + responsive size
<img src="photo-800.webp" srcSet="photo-400.webp 400w, photo-800.webp 800w" loading="lazy" alt="" />
```

## Main thread blocking

JS is single-threaded: a long computation (parsing, heavy loops) freezes the UI. Move heavy work to a **Web Worker**, or chunk it with `requestIdleCallback`/chunking.

```js
// ✅ the heavy computation runs off the main thread
const worker = new Worker('heavy-calc.worker.js');
worker.postMessage(bigDataset);
worker.onmessage = (e) => setResult(e.data);
```

## Too many state updates

Multiple `setState` calls in a row without batching (outside React handlers, e.g. in `setTimeout`/`fetch` callbacks in older React versions) trigger one render per call. React 18+ does *automatic batching* in most cases, but it's worth checking.

```jsx
// ❌ (pre-React 18, outside a handler) triggers 2 renders
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(true);
}, 1000);

// ✅ grouping related state into a single object/reducer fixes the root cause
const [state, setState] = useState({ count: 0, flag: false });
setTimeout(() => setState(s => ({ count: s.count + 1, flag: true })), 1000);
```

## Unoptimized (third-party) components

Heavy UI libraries imported whole instead of tree-shakeable, or components that don't support `memo` internally. See [tree shaking](../frontend-react/README.md) and audit the bundle with bundle-analyzer.

```ts
// ❌ imports the whole library (~70kb) to use one function
import _ from 'lodash';
_.debounce(fn, 300);

// ✅ imports only what's needed, tree-shakeable
import debounce from 'lodash/debounce';
debounce(fn, 300);
```

---
To measure instead of guess: React DevTools **Profiler**, **Lighthouse**, and **bundle-analyzer** — see [frontend-react](../frontend-react/README.md).
