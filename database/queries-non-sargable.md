# Non-sargable queries

**SARGable** = "Search ARGument ABLE": a condition the engine can resolve using an index directly. A **non-sargable** query forces a *full scan* even when an index exists on the column, because the condition prevents using it.

## Common patterns that break the index

### 1. Function on the indexed column

```sql
❌ SELECT * FROM users WHERE UPPER(email) = 'ANA@MAIL.COM';
✅ SELECT * FROM users WHERE email = 'ana@mail.com'; -- normalize the data on write, not on read
✅ CREATE INDEX idx_email_upper ON users(UPPER(email)); -- or a functional index, if it can't be avoided
```

### 2. Wildcard at the start of a LIKE

```sql
❌ SELECT * FROM users WHERE name LIKE '%ana%';  -- can't use a B-tree, searches everywhere
✅ SELECT * FROM users WHERE name LIKE 'ana%';   -- known prefix, does use the index
   -- For real substring search, use full-text search (GIN/tsvector) or trigram (pg_trgm)
```

### 3. Implicit type conversion

```sql
❌ SELECT * FROM orders WHERE order_id = '123';  -- order_id is int, compared to a string, forces a cast
✅ SELECT * FROM orders WHERE order_id = 123;
```

### 4. Arithmetic operation on the column

```sql
❌ SELECT * FROM sales WHERE price * 1.21 > 100;
✅ SELECT * FROM sales WHERE price > 100 / 1.21;  -- move the calculation to the literal, not the column
```

### 5. `OR` across different columns

```sql
❌ SELECT * FROM users WHERE email = 'x@mail.com' OR phone = '123456';
   -- the planner often can't combine two different indexes efficiently
✅ -- Consider a UNION of two separately indexed queries:
   SELECT * FROM users WHERE email = 'x@mail.com'
   UNION
   SELECT * FROM users WHERE phone = '123456';
```

### 6. `NOT IN`, `!=`, `<>`

```sql
❌ SELECT * FROM orders WHERE status != 'cancelled';
   -- low selectivity + negation, usually ends up as a seq scan
```

### 7. `OR IS NULL` combined with other conditions, and date functions

```sql
❌ SELECT * FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024;
✅ SELECT * FROM orders WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

## General rule

If the indexed column appears **inside** a function, cast, or calculation on the left side of the comparison, the engine generally can't use the index — the column needs to be left "bare" and the transformation moved to the literal on the other side.

## How to detect it

```sql
EXPLAIN ANALYZE SELECT ...;
```

Look for a `Seq Scan` where an `Index Scan` would be expected. See [Database Diagnostics](../diagnostics/database.md).

Related: [Indexes](indexes.md).
