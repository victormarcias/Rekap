# Sharding vs Partitioning

Both are strategies for splitting data and scaling beyond what a single node/table can handle, but they operate at different levels.

## Partitioning

Splitting a **large table on the same server/instance** into smaller sub-tables (partitions), transparent to queries. This is **horizontal** partitioning — it splits **rows**; see [Vertical Partitioning](scaling-database.md#vertical-partitioning) for the other dimension, splitting **columns**.

### Types

| Type | Criterion | Example |
|---|---|---|
| **Range** | Range of values | Partitioning `orders` by month (`created_at`) |
| **List** | Discrete values | Partitioning `users` by country (`AR`, `BR`, `MX`) |
| **Hash** | Hash of a column, uniform distribution | Partitioning by `hash(user_id) % 4` |

```sql
-- Postgres: partition by range
CREATE TABLE orders (
  id bigint, created_at date, total numeric
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

**Benefits**: queries that filter by the partition column only touch the relevant partition (*partition pruning*), faster maintenance (`VACUUM`, dropping an entire old partition instead of a row-by-row `DELETE`).

**Still a single server**: it doesn't solve CPU/disk/memory limits of a single instance.

## Sharding

Splitting data **across multiple independent servers/instances** (each shard is a separate database, potentially on a different machine).

### Strategies

- **Key-based (hash) sharding**: `shard = hash(user_id) % N`. Uniform distribution, but resharding (adding a node) is expensive because it changes the mapping for almost every key.
- **Range-based sharding**: shard based on the key's range (e.g. users A-M on shard 1, N-Z on shard 2). Easy to reason about, but can create *hotspots* if traffic isn't evenly distributed.
- **Directory-based sharding**: a lookup table maps each key to its shard. Flexible (allows moving individual keys) but adds a point of indirection/failure.

### Sharding challenges

- **Cross-shard queries**: a `JOIN` between data on different shards isn't native — it has to be resolved in the application layer or with a query router.
- **Distributed transactions**: keeping ACID across shards requires protocols like *two-phase commit* or accepting eventual consistency.
- **Resharding**: adding/removing nodes means redistributing data, a delicate operation in production (mitigated with *consistent hashing*).
- **Foreign keys** across shards can't be guaranteed at the engine level.

## Comparison table

| | Partitioning | Sharding |
|---|---|---|
| Level | One table, one server | Multiple servers |
| Goal | Manageability, maintenance, partition pruning | Scaling horizontally beyond a single server |
| Query transparency | High (the engine handles it) | Low (requires routing logic in the app or a proxy) |
| Solves a single node's hardware limit | ❌ | ✅ |

## In practice

They're combined: each shard can in turn be partitioned internally. Example: 8 Postgres shards, each with the `orders` table partitioned by month.

---
Related: [Normalization](normalization.md) (denormalization often goes hand in hand with sharding to avoid cross-shard joins), [NoSQL](nosql.md) (many NoSQL databases shard natively, e.g. Cassandra, MongoDB), [Database Scalability](scaling-database.md) (vertical partitioning, active vs historical table, snapshot tables).
