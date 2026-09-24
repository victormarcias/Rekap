# Database Scalability

[Sharding vs partitioning](sharding-vs-partitioning.md) already covers in depth how to scale by distributing **data**. This completes the picture with the other strategy: scaling by distributing **reads**.

## Read replicas

Read-only copies of the primary database, synchronized through replication (usually asynchronous). Writes all still go to the primary/master node; reads are distributed across the replicas — in most real apps, reads far outnumber writes, so scaling reads this way solves a big chunk of the load without touching how data is partitioned.

```
                    ┌──────────┐
    writes    ───→ │ Primary  │
                    └────┬─────┘
                         │ asynchronous replication
              ┌──────────┼──────────┐
              ▼          ▼          ▼
         ┌─────────┐┌─────────┐┌─────────┐
 reads →   │Replica 1││Replica 2││Replica 3│
         └─────────┘└─────────┘└─────────┘
```

**The cost**: asynchronous replication implies *replication lag* — a replica can be a few milliseconds (or more, under load) behind the primary. Reading from a replica right after writing to the primary can return data that hasn't replicated yet — the same consistency-vs-availability trade-off from [CAP Theorem](nosql.md#cap-theorem), applied inside a relational database. For the "I need to read exactly what I just wrote" case, you have to deliberately read from the primary, not a replica.

## Processing large results in chunks

Fetching a table with millions of rows with a single `fetchall()` loads the **entire** result into memory at once — the limit is no longer the database, it's the RAM of the process doing the reading (see [Memory Scalability](../devops/scaling-memory.es.md)). The fix isn't fetching less data, it's fetching the same data **in parts**: request a chunk, process it, discard it, request the next one — memory usage stays bounded to the chunk size, no matter how large the full table is.

```python
# ❌ loads the entire result into memory — with a large table, this can OOM the process
rows = cursor.execute("SELECT * FROM huge_table").fetchall()
for row in rows:
    process(row)

# ✅ fetches in chunks: each batch is processed and released before requesting the next
cursor.execute("SELECT * FROM huge_table")
while True:
    chunk = cursor.fetchmany(1000)
    if not chunk:
        break
    for row in chunk:
        process(row)
    # the previous chunk is free for the garbage collector here, before the next fetchmany
```

Many drivers also support a **server-side cursor** (in Postgres, a named cursor): instead of the driver fetching the whole response and handing it out in pieces from the client, the database itself holds the result and streams it — this saves memory on the DB side too, not just the app side. Typical use case: ETL/export processes that walk an entire table without ever needing to hold it fully in memory.

## Vertical Partitioning

Horizontal [partitioning](sharding-vs-partitioning.md#partitioning) splits **rows** (same table, same columns, different partitions); vertical splits **columns**: the table is split into two or more narrower tables, each with a subset of columns, joined by the same primary key.

The typical sign that it's needed: a wide table, with columns that are almost never used together, many of them `NULL` most of the time because they actually store different concepts (contact info, demographics, billing info) crammed into one row.

```sql
-- ❌ one table mixing frequently used data with rarely used/optional data
CREATE TABLE person (
  id bigint PRIMARY KEY,
  first_name text, last_name text,        -- read on almost every query
  additional_contact_info xml,             -- almost always NULL, heavy, rarely used
  demographics xml,                        -- almost always NULL, heavy, rarely used
  credit_card_id bigint                     -- only applies to some records
);

-- ✅ split by columns: the frequent stuff stays light, the heavy/optional stuff is queried separately
CREATE TABLE person (
  id bigint PRIMARY KEY,
  first_name text, last_name text
);
CREATE TABLE person_details (
  person_id bigint PRIMARY KEY REFERENCES person(id),
  additional_contact_info xml,
  demographics xml
);
```

**Why it helps**: a query that only needs `first_name`/`last_name` (most of them) now reads smaller rows — more rows fit in each disk page, more rows fit in the in-memory buffer pool (see [Memory Scalability](../devops/scaling-memory.es.md)) — without having to drag along heavy or optional columns it didn't even ask for.

**The limit**: if the spread isn't "a handful of related column groups" but is genuinely **variable per row** (each record needs a different, unpredictable set of fields), splitting further vertically isn't enough — that's where it's worth evaluating [NoSQL](nosql.md) (a document store) instead of forcing the relational model further.

## Active vs historical table

When most queries only care about "recent" records (open invoices, last month's orders) but the table accumulates years of closed data, moving the old stuff to a separate table (`invoices_history`) keeps the active table small — smaller, faster indexes, cheaper backups/maintenance, and day-to-day queries don't have to wade through years of records they'll never need.

```sql
-- move invoices closed more than 2 years ago to a separate historical table
BEGIN;
INSERT INTO invoices_history SELECT * FROM invoices WHERE created_at < now() - interval '2 years';
DELETE FROM invoices WHERE created_at < now() - interval '2 years';
COMMIT;
```

**Difference from [range partitioning](sharding-vs-partitioning.md#partitioning)**: partitioning is transparent to queries — a `SELECT` on `invoices` keeps working the same no matter how many partitions exist behind it. Splitting into `invoices`/`invoices_history` is an explicit split at the application level: code that needs old data has to *know* the second table exists and query it on purpose. It's simpler to implement (no partitioning config needed in the engine), but less transparent.

## Snapshot tables vs materialized views

Both precompute and persist the result of an expensive aggregation, but they differ in who controls the refresh:

- **Materialized view**: the DB engine refreshes it (`REFRESH MATERIALIZED VIEW`) — it's still a native database object, with its own internal handling.
- **Snapshot table**: a regular table, populated by an external job (cron, worker) that runs the calculation and does `INSERT`/`UPDATE` — full control over when and how it refreshes (e.g. incremental instead of recalculating from scratch), at the cost of maintaining that job yourself.

```sql
-- ✅ snapshot table: a job runs this every night, not the DB itself
CREATE TABLE daily_sales_snapshot (
  date date PRIMARY KEY,
  total numeric
);

-- the job (cron/worker) does this upsert every night — incremental refresh, doesn't recompute everything
INSERT INTO daily_sales_snapshot (date, total)
SELECT current_date, SUM(total) FROM sales WHERE created_at::date = current_date
ON CONFLICT (date) DO UPDATE SET total = EXCLUDED.total;
```

**When to use each one**: a materialized view when the engine already supports the kind of refresh you need, with no extra application code. A snapshot table when you need an incremental refresh, coordinated with other steps in a pipeline, or when the engine doesn't even support native materialized views (MySQL, for example, doesn't have them — snapshot tables fill the gap).

## Vertical vs horizontal, applied to DB

- **Vertical**: more CPU/RAM/IOPS on the same DB instance — the simplest default, with a physical ceiling.
- **Horizontal (reads)**: read replicas, as above.
- **Horizontal (data)**: [sharding](sharding-vs-partitioning.md#sharding) — when neither scaling vertically nor adding read replicas is enough, because the problem is data/write volume, not just reads.

Read replicas and sharding aren't mutually exclusive — a large system typically combines both: several shards, each with its own read replicas.

---
Related: [Sharding vs partitioning](sharding-vs-partitioning.md), [Consistency](../system-design/atributos-de-calidad.md#consistencia), [Connection pooling](../diagnostics/backend.md#poorly-managed-connections), [Memory Scalability](../devops/scaling-memory.es.md), [NoSQL](nosql.md).
