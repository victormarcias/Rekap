# System Quality Attributes ("-ilities")

Non-functional properties that define how good a design is beyond "it works" — most architecture decisions are conscious trade-offs between these attributes, not the maximization of a single one.

## Scalability

A system's ability to handle more load (users, requests, data) by adding resources, without degrading performance or requiring a redesign. The most common blocker: storing state in the process's memory, which ties a user to a specific instance and prevents scaling horizontally without problems.

```js
// ❌ session in process memory: doesn't scale horizontally —
// the next request from that user can land on another instance that doesn't have this session
const sessions = {};
app.post('/login', (req, res) => { sessions[req.body.userId] = { loggedIn: true }; });

// ✅ state in a shared external store (Redis): any instance can handle it
app.post('/login', async (req, res) => {
  await redis.set(`session:${req.body.userId}`, JSON.stringify({ loggedIn: true }));
});
```

See [Vertical vs Horizontal Scalability](../devops/scaling-vertical-vs-horizontal.md) for the concrete infrastructure-level implementation, and [Sharding vs partitioning](../database/sharding-vs-partitioning.md) for the database angle.

## Immutability

Once a piece of data is created, it doesn't change — any "modification" produces a new copy. It matters because it eliminates an entire class of concurrency bugs (nobody can mutate something someone else is reading at the same time — see [Locks](../database/locks.md)), makes state predictable, and allows detecting changes by comparing references instead of doing an expensive deep comparison — it's the basis of how React decides whether to re-render.

```python
# ❌ mutable: any code with a reference to the object can alter it
# without the rest of the app finding out, creating hidden side effects
config = {"retries": 3}
def process(cfg):
    cfg["retries"] = 0

# ✅ immutable: any change produces a new object, the original stays intact
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class Config:
    retries: int = 3

new_config = replace(config, retries=0)  # original config didn't change
```

```ts
// ❌ mutating state directly: React compares by reference and doesn't detect the change (same object)
const [user, setUser] = useState({ name: 'Ana', age: 30 });
user.age = 31; // doesn't trigger a re-render

// ✅ creating a new object: the reference changes, React does re-render
setUser({ ...user, age: 31 });
```

### Shallow copy vs Deep copy

The spread (`{...obj}`), `Object.assign`, `dict.copy()`, or `dataclasses.replace` from the examples above only copy the **first level**. Any nested object or array inside is still the **same reference** as in the original — mutating that inner level breaks the immutability guarantee without it being obvious, because the top level does look "new."

```python
import copy

original = {"user": {"name": "Ana"}, "count": 1}

shallow = original.copy()          # or dict(original), or {**original}
shallow["count"] = 2                # ✅ doesn't affect original — the top level was actually copied
shallow["user"]["name"] = "Beto"     # ❌ this DOES modify original["user"]["name"] — same nested reference

deep = copy.deepcopy(original)
deep["user"]["name"] = "Carla"        # ✅ now it doesn't touch the original at any level
```

```ts
const original = { user: { name: 'Ana' }, count: 1 };

const shallow = { ...original };
shallow.count = 2;              // ✅ doesn't affect original
shallow.user.name = 'Beto';      // ❌ does affect original.user.name — same nested reference

const deep = structuredClone(original); // native deep copy (or JSON.parse(JSON.stringify(x)) as an old alternative)
deep.user.name = 'Carla';         // ✅ doesn't touch the original at any level
```

That's why `setUser({ ...user, age: 31 })` above is safe — `age` is a primitive value, not a nested object. If `user` had `address: { city: 'BA' }`, that same shallow spread wouldn't be enough to modify `city` without mutating the original: you'd need `{ ...user, address: { ...user.address, city: 'Rosario' } }` (or an immutable state management library) to hold the guarantee at deeper levels.

## Availability

The proportion of time the system responds correctly. Measured in "nines" (99.9% ≈ 8.7 hours of downtime a year, 99.99% ≈ 52 minutes). Achieved through redundancy: more than one instance running, in more than one availability zone, with a load balancer that stops sending traffic to the one that fails.

```yaml
# ✅ K8s: without this, a downed pod keeps receiving traffic until someone notices
readinessProbe:
  httpGet: { path: /health, port: 3000 }
  periodSeconds: 10
```

Direct trade-off with **Consistency** (see below) — it's the essence of the CAP theorem, see [NoSQL](../database/nosql.md).

## Consistency

All nodes/readers of the system see the same data at the same time. **Strong consistency**: a write is immediately visible to every subsequent read (simpler to reason about, more expensive to achieve in distributed systems). **Eventual consistency**: a write takes a while to propagate to every node, but converges (more available, cheaper, but a user may see stale data for a moment).

```sql
-- ✅ Postgres, strong consistency within a transaction:
-- no other connection sees the updated balance until the COMMIT
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

See [ACID / isolation levels](../database/acid.md) for strong consistency, and [NoSQL](../database/nosql.md) for the trade-off with availability (CAP theorem).

## Fault tolerance

The system keeps working (even if in degraded mode) when a component fails, instead of falling over in a cascade. The most common pattern: the **circuit breaker** — stops trying against a service that already proved to be down, instead of piling up timeouts that exhaust its own resources.

```ts
// ✅ simplified circuit breaker: cuts off attempts after repeated failures
class CircuitBreaker {
  private failures = 0;
  private open = false;

  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.open) throw new Error('Circuit open — service unavailable, not retrying');
    try {
      const result = await fn();
      this.failures = 0; // recovers after a success
      return result;
    } catch (err) {
      if (++this.failures >= 3) this.open = true; // cuts off after 3 failures in a row
      throw err;
    }
  }
}
```

## Idempotency

Running the same operation multiple times produces the same result as running it once. Critical for safe retries after network failures: if a client sends a request, the connection drops before it gets the response, and it retries — did the server already process the first attempt or not? Without idempotency, that retry can duplicate a charge.

```python
# ❌ not idempotent: retrying after a timeout can charge twice
def charge(amount, card):
    return payment_gateway.create_charge(amount, card)

# ✅ idempotent: the same idempotency_key never generates a second charge,
# the server uses it to detect "I already processed this request before"
def charge(amount, card, idempotency_key):
    return payment_gateway.create_charge(amount, card, idempotency_key=idempotency_key)
```

In HTTP, `GET`/`PUT`/`DELETE` are defined as idempotent by spec; `POST` isn't — that's why `POST` is the riskiest verb to retry automatically without an idempotency key.

## Observability

How well you can understand a system's internal state by looking at its external outputs (logs, metrics, traces), without having to modify it or debug it live. The pillar that gets forgotten most: without a **correlation ID** that travels between services, an individual log entry is useless for reconstructing what happened to a request that crossed 5 microservices.

```js
// ✅ structured log with a correlation id — lets you trace a request across services
logger.info({ event: 'order_created', orderId, correlationId: req.headers['x-correlation-id'] });
```

## Elasticity

A particular case of scalability: the ability to scale automatically both up **and down** based on real-time demand, with no manual intervention. The difference from plain "scalability" is that automation — a system can be scalable (handles more load if you add resources by hand) without being elastic (nobody adds them on its own).

```yaml
# ✅ K8s HPA: adds/removes pods on its own, based on real CPU usage
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 70 } }
```

## Maintainability

How easy it is to understand, modify, and extend the system without breaking things or taking longer and longer with each change. There's no direct "gauge" — it's inferred from proxies: coupled vs decoupled code, tests that give you confidence to refactor without fear, a single business rule living in one place instead of copied in several.

```python
# ❌ the same "magic threshold" repeated in several places —
# changing it means finding and updating every occurrence, with the risk of missing one
def can_checkout(cart_total):
    return cart_total >= 50

def apply_free_shipping(cart_total):
    return cart_total >= 50

# ✅ a single source of truth — changing the business rule is a single place
FREE_SHIPPING_THRESHOLD = 50

def can_checkout(cart_total):
    return cart_total >= FREE_SHIPPING_THRESHOLD
```

[SOLID](solid.md) is, in essence, a set of principles meant to maximize this — the Open/Closed Principle in particular ("open to extension, closed to modification") is the most direct version of "add a new feature without having to touch code that already works and is already tested."

## Performance (Speed)

How fast the system responds — two sides that don't always go together: **latency** (how long a single operation takes) and **throughput** (how many operations per second it can sustain). Optimizing one sometimes worsens the other (e.g. processing in large batches improves throughput but worsens the latency of each individual request waiting for the batch to fill up).

```
p50 (median): 90ms   — half of requests respond faster than this
p95: 320ms
p99: 1800ms            — 1 in every 100 requests takes this long or more

The average (e.g. 150ms) hides the p99: if 1% of your users
suffer 1.8 seconds of latency, the average alone won't show it —
looking at high percentiles is the only way to see that long tail.
```

Measuring it well matters more than any single technique — see [Performance Diagnostics](../frontend-react/performance-diagnostics.md) and [Web Vitals](../frontend-react/web-vitals.md) on the frontend side, [Query Optimization](../database/query-optimization.md) on the database side, and [Backend Diagnostics](../diagnostics/backend.md) / [Frontend Diagnostics](../diagnostics/frontend.md) for the most common causes of a slow system.

## Summary

| Attribute | In one sentence |
|---|---|
| Scalability | Handles more load by adding resources |
| Immutability | Data doesn't change, it gets replaced |
| Availability | The system responds when you need it |
| Consistency | Everyone sees the same data at the same time |
| Fault tolerance | Keeps working even if something breaks |
| Idempotency | Repeating the operation doesn't change the result |
| Observability | You can understand what's happening inside from outside |
| Elasticity | Scales itself, up and down, based on demand |
| Maintainability | Easy to modify without breaking everything else |
| Performance | Responds fast and sustains the load |
| Security | Protects data and access — see [Authentication and Security](../backend/authentication.md) |

These attributes usually pull against each other (the classic example: more Consistency generally costs Availability). Every architecture decision is consciously choosing what to prioritize for the use case — there's no design that maximizes all of them at once.
