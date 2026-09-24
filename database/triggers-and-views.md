# Triggers and Views

## Views

A view is a **named, saved query** — queried as if it were a table, but it stores no data of its own: every time you `SELECT` from it, the DB runs the original query underneath, against the real, current data in the base tables.

```sql
CREATE VIEW pending_orders AS
  SELECT o.id, o.customer_id, o.total, c.name AS customer_name
  FROM orders o
  JOIN customers c ON c.id = o.customer_id
  WHERE o.status = 'pending';

-- queried like a normal table
SELECT * FROM pending_orders WHERE total > 1000;
```

**What it's for**: simplifying complex/repeated queries (hiding a 4-table JOIN behind a simple name), and as a security layer — granting access to a view that only exposes certain columns/rows, without giving access to the full table.

**The limit**: since it persists nothing, a view over an expensive query (aggregations, big joins) is **just as slow as the original query**, every time it's queried — it doesn't speed anything up by itself. That's what the materialized view is for.

## View vs Materialized View

Already covered in detail in [Materialized views](../diagnostics/database.md#materialized-views) and compared against snapshot tables in [Snapshot tables vs materialized views](scaling-database.md#snapshot-tables-vs-materialized-views) — here's the summary of the underlying difference:

| | View | Materialized View |
|---|---|---|
| Stores data | No — runs the query on every read | Yes — persists the result |
| Read performance | Same as the original query | Fast — reads an already-computed result |
| Data freshness | Always up to the second | Stale until the next refresh |
| Cost | None extra (takes no space) | Storage space + refresh cost |

## Triggers

Code the database runs **automatically** when an event occurs on a table (`INSERT`, `UPDATE`, `DELETE`) — unlike a function or stored procedure, a trigger is never called explicitly, it fires on its own.

```sql
-- keeps updated_at in sync without any application having to remember to do it
CREATE FUNCTION set_updated_at() RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_orders_updated_at
  BEFORE UPDATE ON orders
  FOR EACH ROW
  EXECUTE FUNCTION set_updated_at();
```

### `BEFORE` vs `AFTER`

- **`BEFORE`**: runs before the change is applied — can modify the data that's about to be saved (like the example above, which overwrites `NEW.updated_at`) or cancel the operation outright.
- **`AFTER`**: runs after the change has already been applied — typical for side effects that shouldn't alter the data itself, like writing to an audit table.

```sql
-- AFTER: logs the change, without modifying the row that was just inserted
CREATE TRIGGER trg_audit_orders
  AFTER INSERT ON orders
  FOR EACH ROW
  EXECUTE FUNCTION log_order_created();
```

### Trade-off

A trigger guarantees the logic runs **always**, no matter which service or script made the change — not even a developer with direct DB access can accidentally skip it. The cost is the same underlying problem as stored procedures (see [Trade-off: logic in the DB vs in the application](stored-procedures-vs-functions.md#trade-off-logic-in-the-db-vs-in-the-application)), made worse: a trigger is logic that's **invisible** from the application code — someone reading the app's code has no way to know it exists, until they discover it while debugging unexpected behavior. That's why they're used sparingly, typically for data invariants (auditing, timestamps, integrity validations) and not for core business logic.

### Why they're seen less and less in application development

In a modern (web/mobile) application backend, triggers and stored procedures show up less and less — the logic that used to live in the DB is now written directly in the application code using an **ORM** (see [ORM](../backend/controller-service-repository.md#orm-object-relational-mapping)), which already solves much of what a trigger used to (e.g. automatic `updated_at` via an ORM hook, not a SQL trigger) without the downside of being invisible from the code.

Where they're still common is in more **data**-oriented roles (Data Engineer, Data Platform) — ETL pipelines, data warehouse integrity, or legacy systems where the logic has lived there for years and migrating it isn't trivial. Worth knowing they exist and what they're for, but it's not what you'll be writing day to day in a typical FastAPI/Django/Node application backend.

---
Related: [Stored procedures vs functions](stored-procedures-vs-functions.md), [ACID](acid.md#acid) (triggers are part of what guarantees Consistency).
