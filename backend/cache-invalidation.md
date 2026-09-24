# Cache Invalidation

"There are only two hard things in Computer Science: cache invalidation and naming things" — the joke is old because it's true: caching is easy, invalidating at the right moment without serving stale data or dumping the whole cache more than needed, is the hard part.

## TTL vs explicit invalidation

- **TTL (Time To Live)**: the entry expires on its own after X seconds. Simple, doesn't require anyone to remember to invalidate anything — but during the TTL window, a real change might not be reflected yet (stale data by design).
- **Explicit invalidation**: the code that writes the data also deletes/updates the corresponding cache entry at the same moment. More precise (zero stale-data window), but requires remembering to invalidate in **every** place that can modify that data — a single place that forgets leaves the cache out of date indefinitely.

```python
def update_product(id, data):
    db.products.update(id, data)
    cache.delete(f"product:{id}")  # ✅ explicit invalidation in the same write
```

## Cache-aside, write-through, write-behind

The **cache-aside** pattern (read from cache, if it's missing go to the DB and cache the result) is already covered with a code example in [Backend Diagnostics](../diagnostics/backend.md#missing-cache). The other two strategies write differently:

- **Write-through**: every write goes to the cache first, and the cache takes care of propagating it to the DB synchronously. The cache never goes stale, but every write pays the latency of both.
- **Write-behind (write-back)**: the write goes to the cache and is confirmed immediately; propagation to the DB happens in the background, asynchronously. Faster writes, but there's a window where the data only exists in the cache — if the cache goes down before propagating, it's lost.

## Cache stampede (thundering herd)

A heavily requested key expires (TTL) and, at that same instant, hundreds of concurrent requests ask for it — they all see a cache miss at once and hit the DB simultaneously, as if the cache didn't exist. The spike can take down the DB at the worst possible moment (the key was popular, that's why it was cached).

**Mitigations**:
- *Locking*: the first request that detects the miss recomputes and caches it; the rest wait for that result instead of all going to the DB.
- *Early recomputation*: recompute the key a bit before it expires, so there's never a window without cache.

```python
# ✅ simple lock: only one request recomputes, the rest wait
def get_product(id):
    cached = cache.get(f"product:{id}")
    if cached:
        return cached
    with lock(f"lock:product:{id}"):          # other requests wait here
        cached = cache.get(f"product:{id}")    # re-check: maybe someone else cached it while waiting
        if cached:
            return cached
        result = db.products.find(id)
        cache.set(f"product:{id}", result, ttl=300)
        return result
```

---
Related: [Backend Diagnostics](../diagnostics/backend.md#missing-cache) (cache-aside), [Consistency](../system-design/atributos-de-calidad.md#consistencia).
