# Backend Diagnostics

Most common causes of server-side slowness, from most to least frequent.

## N+1 problem

For each item in a collection, an extra query/request fires instead of fetching everything in one operation. Classic in ORMs with lazy loading.

```js
// ❌ 1 query for orders + N queries, one per order, for its user
const orders = await Order.findAll();
for (const o of orders) { o.user = await User.findById(o.userId); }

// ✅ a single query with join/eager loading
const orders = await Order.findAll({ include: User });
```

## Missing cache

Recomputing or refetching data that rarely changes (config, catalogs, expensive query results) on every request. Add an in-memory cache (Redis) with proper invalidation — see [system-design](../system-design/README.md).

```js
// ✅ Node + Redis: cache-aside pattern
async function getProduct(id) {
  const cached = await redis.get(`product:${id}`);
  if (cached) return JSON.parse(cached);

  const product = await db.products.findById(id);
  await redis.set(`product:${id}`, JSON.stringify(product), 'EX', 300); // TTL 5 min
  return product;
}
```

## Clustering

A single Node/Python process doesn't use all of the machine's cores. Use the `cluster` module, PM2 in cluster mode, or multiple workers/gunicorn to scale vertically within the same server.

```js
// ✅ Node: one worker per core
import cluster from 'node:cluster';
import os from 'node:os';

if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
} else {
  startServer(); // each worker runs its own instance of the server
}
```

## Memory

Memory leaks (listeners never removed, unbounded caches, closures holding onto references) degrade performance over time and eventually force swapping or OOM crashes. Take two **heap snapshots** (e.g. Chrome DevTools → Memory, or `node --inspect`) separated by a period of normal traffic and compare them: if an object type keeps growing between snapshots and never drops, that's the one leaking — the snapshot tells you *which* object is holding onto memory, not just how much memory there is.

```js
// ❌ leak: the listener is never removed, they pile up on every request
function handleRequest() { emitter.on('data', onData); }

// ✅ remove it when done
function handleRequest() {
  emitter.on('data', onData);
  emitter.once('end', () => emitter.off('data', onData));
}
```

## CPU bound

Compute-intensive operations (compression, cryptography, image processing) block the event loop in single-threaded runtimes (Node). Delegate to workers, background job queues, or separate services.

```js
// ✅ Node: worker_threads takes the heavy computation off the main event loop
import { Worker } from 'node:worker_threads';
const worker = new Worker('./resize-image.worker.js', { workerData: { path } });
```

## HTTP chaining

An endpoint that internally makes multiple sequential HTTP calls to other services, stacking up latencies instead of parallelizing them (`Promise.all`) or resolving them with a single aggregated request.

```ts
// ❌ 300ms of accumulated latency (100ms each request, sequential)
const user = await fetchUser(id);
const orders = await fetchOrders(id);
const reviews = await fetchReviews(id);

// ✅ 100ms total, in parallel
const [user, orders, reviews] = await Promise.all([
  fetchUser(id), fetchOrders(id), fetchReviews(id),
]);
```

## Unnecessary dependencies

Calling an external service, loading a heavy library, or running a query whose result isn't used. Every dependency adds latency and points of failure — audit what's actually needed in the hot path.

```py
# ❌ imports the whole library for a single calculation
import pandas as pd
avg = pd.Series(values).mean()

# ✅ statistics (stdlib) is enough, no heavy dependency needed
from statistics import mean
avg = mean(values)
```

## Unoptimized algorithms

High algorithmic complexity (`O(n²)` where `O(n log n)` or `O(n)` would do) over large collections. Before optimizing infrastructure, check whether the algorithm is the problem.

```py
# ❌ O(n²): scans the list for every element
duplicates = [x for x in items if items.count(x) > 1]

# ✅ O(n): uses a set/dict for O(1) lookup
seen, duplicates = set(), set()
for x in items:
    (duplicates if x in seen else seen).add(x)
```

## Not releasing system resources

File handles, sockets, or buffers left open without being closed (missing `finally`/`using`/context managers) exhaust process resources over time.

```py
# ❌ if something fails before close(), the file handle stays open
f = open('data.csv')
process(f)
f.close()

# ✅ context manager: always closes, even on an exception
with open('data.csv') as f:
    process(f)
```

## Parallelism and concurrency

Independent work resolved sequentially when it could run in parallel (`await` one by one instead of `Promise.all`), or the opposite: uncontrolled parallelism that saturates CPU/connections.

```py
# ✅ Python: run independent tasks in parallel with asyncio
import asyncio
results = await asyncio.gather(fetch_a(), fetch_b(), fetch_c())
```

## Poorly managed connections

Not using **connection pooling** to the database (opening a new connection per request is expensive), or not releasing connections back to the pool after use, exhausting the pool under load.

```js
// ✅ reusable pool, not a new connection per request
const pool = new Pool({ max: 20 });
```

---
To confirm before optimizing: runtime profiler (`node --prof`, `py-spy`), APM (New Relic/Datadog), and per-endpoint latency logs.
