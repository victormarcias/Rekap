# Stored Procedures vs Functions

Both are SQL/procedural code stored and executed on the database server side, but they have key differences.

## Main differences

| | Stored Procedure | Function |
|---|---|---|
| Invoked with | `CALL proc(...)` | `SELECT func(...)` — inside a query |
| Return | Optional, can return multiple result sets or none | Mandatory, returns a value (scalar, table, etc.) |
| Transaction control | Can do its own `COMMIT`/`ROLLBACK` (depending on the engine) | Can't control transactions — runs inside the caller's transaction |
| Use in SELECT | Can't be used inside a `SELECT`/`WHERE` | Yes, can be used in `SELECT`, `WHERE`, `JOIN` |
| Side effects | Meant to have side effects (INSERT/UPDATE/DELETE, business logic) | Should preferably be pure/deterministic, though some engines allow side effects |

## Example — Function (Postgres)

```sql
CREATE FUNCTION order_total(order_id int) RETURNS numeric AS $$
  SELECT SUM(price * quantity) FROM order_items WHERE order_id = order_id;
$$ LANGUAGE sql;

SELECT id, order_total(id) FROM orders; -- usable inside a SELECT
```

## Example — Stored Procedure (Postgres)

```sql
CREATE PROCEDURE archive_old_orders() AS $$
BEGIN
  INSERT INTO orders_archive SELECT * FROM orders WHERE created_at < now() - interval '2 years';
  DELETE FROM orders WHERE created_at < now() - interval '2 years';
  COMMIT; -- a procedure can manage its own transaction
END;
$$ LANGUAGE plpgsql;

CALL archive_old_orders();
```

## When to use each one

- **Function**: reusable calculations within queries (e.g. computing a total, validating a format, transforming a value).
- **Procedure**: batch processes, administrative tasks, logic involving multiple steps with its own transaction control (ETL, scheduled jobs).

## Trade-off: logic in the DB vs in the application

**In favor of stored procedures/functions:**
- Fewer network round-trips (the logic runs where the data lives).
- Useful when several services/languages need the same logic consistently.

**Against:**
- Hard to version, test, and debug compared to application code (less tooling, no type-checking from the main language).
- Couples business logic to the specific database engine, complicating migrations.
- The modern trend (microservices, cloud-native architectures) prefers keeping business logic in the application layer and using the DB only for persistence — except for specific performance or critical-integrity cases.
