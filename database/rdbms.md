# RDBMS (Relational Database Management System)

## What it is

Software that organizes data into **tables** (rows and columns) related to each other through keys (foreign keys), with **SQL** as the query language and **ACID** guarantees for transactions. Implements the relational model (Edgar Codd's theory, 1970). Examples: PostgreSQL, MySQL, SQL Server, Oracle — each with its own engine underneath, but the same data model on top.

## When to use it (and when not to)

**Use it when:**
- The data has clear relationships between entities and integrity matters (an order can't exist without a valid customer — a foreign key guarantees that, not application code).
- Transactions need to be atomic and consistent (money, inventory, reservations — all or nothing, never a half-applied state).
- Queries need complex joins and aggregations over structured data.
- The schema is relatively stable — it doesn't change shape every day.

**Don't use it (or not only) when:**
- The access pattern is simple and known in advance (lookups by id, no joins) and the priority is scaling horizontally without limit — [NoSQL](nosql.md) fits better there.
- The schema changes constantly and forcing a fixed structure costs more than it's worth.

## How it works internally

A SQL query goes through several layers before returning a result:

```
Client (SQL) → Parser → Query Optimizer → Execution Engine → Storage Engine → Disk
                              ↑                    ↓
                        picks the cheapest   uses Indexes to avoid
                        plan                 scanning everything
```

1. **Parser**: validates that the SQL is syntactically correct.
2. **Query Optimizer**: among several possible ways to run the query, picks the one it estimates is cheapest — using table statistics (see [Planner statistics](../diagnostics/database.md#planner-statistics)) and deciding whether an [index](indexes.md) is worth it or scanning the whole table.
3. **Execution Engine**: runs the chosen plan, reading/writing rows.
4. **Storage Engine**: doesn't read/write row by row straight to disk — it works in **pages** (blocks of several KB), and keeps a **buffer pool** in memory with the most recently used pages, to avoid hitting disk on every operation (the same principle as a cache — see [Backend Diagnostics](../diagnostics/backend.md#missing-cache) for the general caching concept).

## Write-Ahead Log (WAL) — how nothing gets lost if the server crashes

Before modifying the real data on disk, the change is first written to a **sequential log** (the WAL). Only after that log confirms the change is it applied to the real data pages — which can sit in memory (buffer pool) for a while before syncing to disk, because writing to disk on every transaction would be extremely expensive.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT; -- at this point the WAL has already confirmed the change; the data page can sync to disk later
```

If the server crashes between the `COMMIT` and the page reaching disk, on restart the engine **replays the WAL** from the last safe point — recovering everything that was confirmed, without having had to sync every page on every transaction. This is the concrete mechanism behind the **D** (Durability) in [ACID](acid.md#acid): the guarantee doesn't come from always writing everything to disk, it comes from this log, which is written safely and sequentially (much cheaper than random writes scattered across the data file).

---
Related: [ACID / transactions / isolation levels](acid.md), [Indexes](indexes.md), [Query Optimization](query-optimization.md), [Locks](locks.md), [NoSQL](nosql.md).
