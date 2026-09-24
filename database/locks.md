# Locks (shared/exclusive/MVCC)

Concurrency control mechanisms: how the database keeps simultaneous transactions from stepping on each other's data.

## Race condition — the underlying problem

Happens when two or more operations access the same shared data at the same time, and the final result depends on the exact **order/timing** they interleave in — with no guarantee of which one wins. The classic case is the *lost update*: two requests read the same value, each computes its change based on that read, and whichever writes second **overwrites** the first one's change without ever knowing it existed.

```sql
-- T1 and T2 run "at the same time," both start by reading the same stock
-- T1: SELECT stock FROM products WHERE id = 1;  → reads 5
-- T2: SELECT stock FROM products WHERE id = 1;  → reads 5 (hasn't seen T1's change yet)

-- T1: UPDATE products SET stock = 4 WHERE id = 1;  -- 5 - 1, based on what it read
-- T2: UPDATE products SET stock = 4 WHERE id = 1;  -- 5 - 1, based on what it read

-- result: stock = 4, but 2 units should have sold → real stock = 3
-- T2's UPDATE overwrote T1's without either one knowing about the other
```

## Critical section — the general concept behind the solution

The chunk of code that touches the shared data (the `SELECT` + `UPDATE` above) is a **critical section**: any stretch that can't run in more than one thread/process at a time without risking a race condition. The classic primitives for protecting it are the **mutex** (mutual exclusion — one access at a time) and the **semaphore** (allows up to N simultaneous accesses; a mutex is, at bottom, a semaphore with N=1).

```python
import threading

lock = threading.Lock()  # mutex: mutual exclusion, one thread at a time

def decrement_stock():
    with lock:  # critical section: no one else gets in until this block finishes
        stock[product_id] -= 1
```

The locks in this file (pessimistic/optimistic, shared/exclusive, below) are **one particular case** of this concept — implementing "protect a critical section" at the database level. It's not the only one: a *distributed lock* (e.g. with Redis) protects the same idea, but coordinating across processes/instances instead of across a DB's transactions; and the `threading.Lock()` above protects it within a single process, with no database involved at all. Everything that follows in this file is a way of solving this specifically for data in a DB: either concurrent access is blocked (pessimistic locking, shared/exclusive locks), or the conflict is detected at write time and the overwriting write is rejected (optimistic locking).

## Pessimistic vs Optimistic locking

- **Pessimistic**: assume there's going to be a conflict, so lock the resource before touching it (`SELECT ... FOR UPDATE`). Other processes wait.
- **Optimistic**: assume there won't be a conflict, let everyone read/write freely, and check at write time (e.g. a `version` column) whether someone else modified the data in the meantime — if so, reject the update.

```sql
-- Optimistic locking with a version column
UPDATE products SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 5;
-- If affected rows = 0 → someone else modified it, needs a retry
```

## Shared lock (S) vs Exclusive lock (X)

| Lock | Lets others read | Lets others write | Used for |
|---|---|---|---|
| **Shared (S)** | Yes (another S) | ❌ | `SELECT ... FOR SHARE` |
| **Exclusive (X)** | ❌ | ❌ | `UPDATE`, `DELETE`, `SELECT ... FOR UPDATE` |

```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- exclusive lock on the row
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT; -- releases the lock
```

## Row-level vs table-level

- Most modern engines (Postgres, MySQL InnoDB) lock at the **row level** by default — much better for concurrency than locking the whole table.
- `LOCK TABLE` exists for specific cases (e.g. migrations, batch operations) but blocks everyone else.

## Deadlocks

Two transactions mutually block each other, each waiting for a lock the other holds.

```
T1: locks row A, waits on row B
T2: locks row B, waits on row A
→ deadlock
```

The engine detects the cycle and aborts one of the two transactions (the victim gets an error and has to retry). **Mitigation**: always lock resources in the same order throughout the application (e.g. always by ascending `id`).

## MVCC (Multi-Version Concurrency Control)

Instead of blocking reads, Postgres and MySQL InnoDB keep **multiple versions** of each row:

- Each transaction sees a consistent *snapshot* of the data according to its isolation level, without blocking writers.
- An `UPDATE` doesn't overwrite the row in place: it creates a new version and marks the old one as obsolete (Postgres), or moves it to the *undo log* (MySQL InnoDB).
- A cleanup process (`VACUUM` in Postgres) removes old versions nobody needs anymore.

**Practical consequence**: with MVCC, readers never block writers or vice versa (`SELECT` doesn't wait on an in-progress `UPDATE`) — only writer-vs-writer generates real contention.

See also [ACID / transactions / isolation levels](acid.md), which depends directly on these mechanisms.
