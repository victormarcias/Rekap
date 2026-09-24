# Sync vs Async

FastAPI accepts both `def` and `async def` on a route — the choice isn't cosmetic, it has real performance consequences if done wrong. This isn't `microservice-endpoints.md`'s theory; it's specifically about when async helps, when it does nothing, and what breaks when migrating an entire app from sync to async.

## 1. What the event loop is and why it matters

`asyncio` runs on **a single thread** with cooperative multitasking: a coroutine yields control (with `await`) at every wait point, and the event loop uses that pause to handle other coroutines. If a coroutine never yields (does blocking work with no `await`), it **freezes the entire loop** — nothing else runs until it finishes, not even other users' requests.

```python
import asyncio, time

async def bad_task():
    time.sleep(5)  # ❌ blocks the entire event loop — nothing else runs during these 5 seconds

async def good_task():
    await asyncio.sleep(5)  # ✅ yields control, the loop handles other coroutines meanwhile
```

## 2. `async def` vs `def` on a route — what FastAPI does differently

An `async def` route runs directly on the event loop, on the same thread as everything else. FastAPI automatically sends a regular `def` route to a **separate threadpool** — that's why blocking sync code inside it doesn't jam the rest of the app, at the cost of using a pool thread instead of the main loop.

```python
# ✅ regular def: FastAPI runs it in a threadpool automatically,
# the blocking code inside doesn't freeze the main event loop
@app.get("/legacy")
def get_legacy_data():
    return sync_blocking_call()

# ✅ async def: runs on the shared event loop — only makes sense
# if EVERYTHING inside it is genuinely async (with await)
@app.get("/modern")
async def get_modern_data():
    return await async_non_blocking_call()
```

## 3. I/O-bound vs CPU-bound — the real rule

Async shines at **I/O-bound** work (a network call to another service, a DB query, reading a file): the coroutine spends most of its time waiting, and during that wait the event loop handles other requests. In **CPU-bound** work (parsing a giant JSON, resizing images, a heavy loop) there's no "wait" to yield — a single thread doing computation blocks regardless of `async`/`await`. See [Backend Diagnostics](../../diagnostics/backend.md) (CPU bound) for how to take that work off the main process.

## 4. When async doesn't help (or makes things worse)

The most common trap: declaring `async def` but calling a blocking sync function inside **without** `await`. That releases nothing — it blocks the event loop for **all** concurrent requests, worse than if that same route had been a plain `def` (which FastAPI would have sent to a threadpool instead of jamming the main loop).

```python
# ❌ the worst case: async def calling blocking sync code with no await —
# freezes the event loop for ALL requests, not just this one
@app.get("/orders")
async def get_orders():
    return requests.get("http://other-service/orders")  # sync library, no await

# ✅ if the internal work is sync, use plain def — FastAPI sends it to a threadpool on its own
@app.get("/orders")
def get_orders():
    return requests.get("http://other-service/orders")

# ✅ or use the async version of the HTTP library
@app.get("/orders")
async def get_orders():
    async with httpx.AsyncClient() as client:
        return await client.get("http://other-service/orders")
```

## 5. Async SQLAlchemy: engine, `AsyncSession`, `aiosqlite`

For DB queries to be genuinely non-blocking you need an **async driver** (`aiosqlite` for SQLite, `asyncpg` for Postgres) and `create_async_engine` + `AsyncSession` — it's not enough to wrap the usual sync `Session` in an `async def`, that would still block the loop on every query.

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

engine = create_async_engine("sqlite+aiosqlite:///./app.db")  # async driver, not the usual sync one
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session

@app.get("/orders/{order_id}")
async def get_order(order_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Order).where(Order.id == order_id))
    return result.scalar_one_or_none()
```

## 6. Eager loading relations in async mode

Classic lazy loading (accessing `order.items` only when the attribute is read, firing a new query at that moment) directly **fails** in async SQLAlchemy with a `MissingGreenlet` error — loading the relation there would require doing I/O synchronously outside an `await` context, and the async driver doesn't allow it. The fix is to load the relation upfront with `selectinload`/`joinedload` in the original query.

```python
from sqlalchemy.orm import selectinload

# ❌ lazy loading: accessing order.items after the original query —
# in async mode this fails with MissingGreenlet, it's not just "slower"
result = await db.execute(select(Order).where(Order.id == order_id))
order = result.scalar_one()
print(order.items)  # 💥 error

# ✅ eager loading: everything is fetched in the original query, no access needed afterward
result = await db.execute(
    select(Order).options(selectinload(Order.items)).where(Order.id == order_id)
)
order = result.scalar_one()
print(order.items)  # ✅ already loaded in memory
```

## 7. Exception handlers in async routes

Exception handlers can (and should, if they do I/O) also be `async def`. If a handler logs to an external service or the DB, it needs the same care as point 4: if that call is sync with no `await`, it blocks the event loop just as it would inside a route.

```python
@app.exception_handler(OrderNotFoundError)
async def order_not_found_handler(request: Request, exc: OrderNotFoundError):
    await audit_log.record(f"Order {exc.order_id} not found")  # real I/O → needs await
    return JSONResponse(status_code=404, content={"error": "order_not_found"})
```

## 8. How to measure whether anything actually improved

Don't assume async = faster. The real way to confirm it is comparing throughput and latency under concurrent load, before and after migrating — not in a local single-user script, where sync and async feel identical.

```bash
# ✅ compare requests/sec and p95 latency under real load, don't guess
hey -n 1000 -c 50 http://localhost:8000/orders
```

If an endpoint has no real I/O to wait on (a trivial query to a local DB, for example), migrating it to async only adds the complexity of maintaining an async driver/engine, with no measurable gain.

---
Related: [Microservice Endpoints](microservice-endpoints.md), [Backend Diagnostics](../../diagnostics/backend.md) (CPU bound, parallelism and concurrency).
