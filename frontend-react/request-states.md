# Request States

Any component that displays data coming from an API has to handle **4 states**, not just "the data arrived":

1. **Loading / Pending** — the request is in flight, no response yet.
2. **Success / Data** — it resolved fine, you have the data to display.
3. **Error** — it failed (a network error, or a status code that isn't 2xx — see [HTTP Status Codes](../system-design/http-status-codes.md)).
4. **Retry** — what to do when it fails: retry automatically? how many times? with what wait between attempts?

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);       // state 3: error
  const [retryCount, setRetryCount] = useState(0); // state 4: how many times it already retried

  useEffect(() => {
    setLoading(true);
    setError(null);

    fetch(url)
      .then(r => {
        if (!r.ok) throw new Error(`Status ${r.status}`);
        return r.json();
      })
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url, retryCount]);   // retryCount in the deps: changing it triggers a new attempt

  const retry = () => setRetryCount(c => c + 1);

  return { data, loading, error, retry };
}
```

```jsx
function UserProfile({ userId }) {
  const { data, loading, error, retry } = useFetch(`/api/users/${userId}`);

  if (loading) return <Spinner />;
  if (error) return <button onClick={retry}>Failed to load — retry</button>;
  return <div>{data.name}</div>;
}
```

**Why it's missing so often**: it's easy to cover Loading and Success (the two "happy" cases), and fall short on Error and Retry — the result is a UI that silently hangs or shows a dead-end error when the network fails, instead of giving the user a way to retry.

**Automatic vs manual retries**: retrying automatically (with no limit) can turn into a loop that never stops — same problem, same fix mechanism we already saw with [Circuit Breaker](../system-design/quality-attributes.md#fault-tolerance): cut off after N failed attempts, instead of retrying forever.

This is, at bottom, the problem libraries like React Query or SWR solve — they model these 4 states (plus cache, revalidation, retries with backoff) so you don't have to reimplement them by hand in every custom `useFetch`.

---
Related: [Hooks](hooks.md) (`useEffect`, the example `useFetch`), [HTTP Status Codes](../system-design/http-status-codes.md), [Circuit Breaker](../system-design/quality-attributes.md#fault-tolerance), [Error Boundaries](error-boundaries.md) (render errors, not network errors — a different concept).
