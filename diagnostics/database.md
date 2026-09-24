# Database Diagnostics

Most common causes of database-level slowness, from most to least frequent.

## Misused `WHERE` clauses

Conditions that prevent index usage (functions applied to the column, a leading wildcard in a `LIKE`, implicit casts). See [Non-sargable queries](../database/queries-non-sargable.md).

## Locking and contention (transactions)

Long or poorly isolated transactions hold locks and make others wait — deadlocks, transactions running unnecessary `SELECT`s inside the lock, a stricter isolation level than needed. See [Locks](../database/locks.md) and [ACID / isolation levels](../database/acid.md).

## Overly complex queries (extra joins, `WHERE` with `LIKE`)

Joins that pull in columns/tables that aren't used, or `LIKE '%text%'` filters that force a full scan instead of using full-text search. Simplify the query down to what's actually needed.

## Lack of maintenance (reindexing and statistics)

Fragmented indexes and stale statistics make the planner take bad decisions. `ANALYZE` recomputes the per-column value distribution the planner uses to estimate how many rows a filter will return — without it, it can underestimate the cost of a `Seq Scan` and discard an index that would have been worth using. `VACUUM` reclaims the space left by "dead" rows from MVCC after updates/deletes (see [Locks](../database/locks.md)); without it, the table and its indexes bloat and every scan reads more pages than necessary. Reindexing is needed when an index gets fragmented from heavy writes/deletes and is no longer balanced. Run `VACUUM ANALYZE` (Postgres) / `ANALYZE TABLE` (MySQL) periodically instead of waiting for the problem to show up.

## Indexes

Missing, redundant, or poorly designed (column order in composites, low selectivity). See [Indexes](../database/indexes.md).

## Execution plan

`EXPLAIN ANALYZE` to see whether the planner uses a `Seq Scan` where it should use an `Index Scan`, and where the real cost is (not always where you'd assume).

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
```

## Planner statistics

The optimizer picks a plan based on data-distribution statistics (cardinality, histograms). If they're stale, it chooses suboptimal plans even when the index exists.

**Histogram**: how the planner summarizes a column's value distribution — it splits the range of values into intervals (*buckets*) and stores how many rows fall into each one. Without this, the planner would have to assume values are spread evenly, which is almost never true in real data.

```sql
-- Postgres: see the histogram ANALYZE computed for a column
SELECT histogram_bounds FROM pg_stats WHERE tablename = 'orders' AND attname = 'status';
```

Thanks to the histogram, the planner knows `WHERE status = 'completed'` will return a huge number of rows (a `Seq Scan` beats an `Index Scan` there), while `WHERE status = 'refunded'` returns few (there the index is worth it) — same column, same index, but a different plan depending on which value is queried, because the planner knows how the data is distributed instead of assuming a uniform distribution.

## Materialized views

Precompute and persist the result of an expensive query (aggregations, heavy joins), at the cost of having to refresh them. Useful for dashboards/reports that don't need second-by-second data.

```sql
CREATE MATERIALIZED VIEW sales_by_month AS
SELECT date_trunc('month', created_at) AS month, SUM(total) FROM orders GROUP BY 1;

REFRESH MATERIALIZED VIEW sales_by_month;
```

## OLAP cubes

For heavy multidimensional analytics (BI, historical reports), separating the analytical workload (OLAP) from the transactional one (OLTP) keeps reporting queries from competing for resources with production traffic. See [system-design](../system-design/README.md).

**Classic case: finance.** An OLAP cube lets you "slice" the same revenue across multiple dimensions at once (month, region, product, currency) without writing a new query for every combination — the reason OLAP was born in that world: financial reports built once and navigated from many angles.

## Sharding across separate DBs (horizontal scalability)

Once you've already indexed, cached, and partitioned well, but a single server still can't keep up on CPU, memory, or IOPS, the next scale-out step is distributing data across multiple independent instances (shards). The cost: joins and transactions that used to be native now cross different servers, and have to be resolved by hand in the application layer. See [Sharding vs partitioning](../database/sharding-vs-partitioning.md).

## Range partitioning

A specific case of partitioning where the criterion is a range of values, typically dates (`orders_2024`, `orders_2025`). Useful specifically when queries almost always filter by that range (e.g. "last month's orders"): the planner can directly discard partitions that don't apply (*partition pruning*) instead of scanning the whole table, and it lets you delete old data by dropping an entire partition instead of a row-by-row mass `DELETE`. See [Sharding vs partitioning](../database/sharding-vs-partitioning.md).

## Specific hardware

Before assuming the problem is the query or the index, rule out the physical floor: slow disks (HDD vs SSD/NVMe) directly impact read/write I/O, too little RAM limits how much of the buffer pool/cache lives in memory (more *cache misses* → more disk), and insufficient CPU bottlenecks compute-heavy queries (aggregations, large sorts). An `EXPLAIN ANALYZE` with a good plan but high times usually points here.

## NoSQL (if needed)

If, after indexing, caching, and partitioning the relational database well, it still can't keep up, and the access pattern is simple (key lookups, no complex joins or need for strong transactional integrity), it's worth evaluating moving that specific slice of the domain to NoSQL instead of continuing to force the relational model. It's not a general replacement — it's a tool for the case where the bottleneck is horizontal scale, not relationships. See [NoSQL](../database/nosql.md).
