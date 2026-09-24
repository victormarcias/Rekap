# Indexes

An auxiliary data structure that lets you find rows without scanning the entire table (avoids a *full table scan*).

## Why they exist

Without an index, `WHERE email = 'x@mail.com'` is O(n): it scans every row. With a B-tree index on `email`, the search is O(log n).

```sql
CREATE INDEX idx_users_email ON users(email);
```

## Most common types

| Type | Typical use | Engine |
|---|---|---|
| **B-tree** | Default. Equality, ranges (`<`, `>`, `BETWEEN`), `ORDER BY` | All |
| **Hash** | Equality only (`=`), no good for ranges | Postgres, MySQL |
| **GIN** | Full-text search, arrays, JSONB | Postgres |
| **GiST** | Geometric data, ranges, proximity search | Postgres |
| **Bitmap** | Low-cardinality columns (few distinct values) combined | Oracle, data warehouses |

## Composite indexes

```sql
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

- Column order matters: this index serves `WHERE user_id = ?` and `WHERE user_id = ? AND created_at > ?`, but it does **not** efficiently serve `WHERE created_at > ?` alone (the *leftmost prefix* rule).

## Covering index

An index that includes every column the query needs, letting it answer without touching the table (*index-only scan*):

```sql
CREATE INDEX idx_orders_covering ON orders(user_id) INCLUDE (status, total);
```

## Clustered vs non-clustered

- **Clustered**: the table's data is physically stored in the index's order (e.g. the PK in SQL Server/MySQL InnoDB). There can only be one per table.
- **Non-clustered**: a separate structure that points to the row's location. There can be several.
- Postgres doesn't have a real clustered index (it uses a *heap* + indexes that point to the heap; `CLUSTER` physically reorders once, it doesn't maintain it).

## Cost: they're not free

- Every index speeds up reads but **slows down writes** (`INSERT`/`UPDATE`/`DELETE` also have to keep the index updated).
- They take up disk space.
- Indexing columns used in `JOIN`s (typically foreign keys) speeds up that operation for the same reason it speeds up a `WHERE`: the engine looks it up by index instead of scanning the other side of the join in full.

**"Indexing everything just in case" isn't the solution** — it's the opposite mistake to indexing nothing. Every extra index:
- Adds its own write cost, and that cost **accumulates**: a table with 5 indexes pays the maintenance cost of all 5 on every `INSERT`, not just the most expensive one.
- Can end up **redundant**: an index on `(user_id)` is unnecessary if one already exists on `(user_id, created_at)` — the composite one already covers queries that only filter by `user_id` (see [composite indexes](#composite-indexes) and the *leftmost prefix* rule).
- Is useless if the column has **low cardinality** (e.g. a `boolean`, or a `status` with 3 possible values) — the engine may decide scanning the whole table is cheaper than using that index, unless there's a [partial index](#partial-index-postgres) on the actually selective subset.

The practical rule is to index according to **real query patterns** (looking at what's used in `WHERE`/`JOIN`/`ORDER BY` in production, not "just in case"), and confirm with [`EXPLAIN ANALYZE`](#how-to-tell-if-its-being-used) that the index is actually being used before calling it good.

## Partial index (Postgres)

```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

Useful when only a subset of rows is queried frequently.

## How to tell if it's being used

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'x@mail.com';
```

Look for `Index Scan` / `Index Only Scan` in the plan (vs `Seq Scan`). See [Database Diagnostics](../diagnostics/database.md) for the details of `EXPLAIN ANALYZE`.

Related: [Non-sargable queries](queries-non-sargable.md) — query patterns that prevent the engine from using the index even when it exists.
