# Rollback / Savepoints

## Rollback

Undoes every change made in the current transaction, returning to the state before `BEGIN`.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- realize something's wrong
ROLLBACK; -- the UPDATE never happened
```

- An automatic `ROLLBACK` also happens if the connection drops or if a statement inside the transaction fails (depends on the engine: Postgres aborts the whole transaction on any error until the explicit `ROLLBACK`; MySQL tends to be more lenient).

## Savepoints

Checkpoints **inside** a transaction that let you undo just a part of it, without losing everything done up to that point.

```sql
BEGIN;
INSERT INTO orders (id, user_id) VALUES (1, 10);

SAVEPOINT before_items;
INSERT INTO order_items (order_id, product_id) VALUES (1, 999); -- invalid product_id, fails

ROLLBACK TO SAVEPOINT before_items; -- undoes only the items INSERT, keeps the order

INSERT INTO order_items (order_id, product_id) VALUES (1, 5); -- retry with correct data
COMMIT;
```

- `RELEASE SAVEPOINT before_items` — releases the savepoint without undoing anything (you can no longer go back to it).
- Useful for simulating "nested transactions" (SQL doesn't support them natively) or handling partial errors in batch processes without losing all the prior work.

## Typical use case: bulk import

```sql
BEGIN;
-- for each row of the CSV:
SAVEPOINT row_start;
INSERT INTO products (...) VALUES (...);
-- if it fails, ROLLBACK TO row_start and log the error, move on to the next row
-- if all OK, RELEASE SAVEPOINT row_start
COMMIT; -- at the end, every valid row is persisted
```

Related: [ACID / transactions / isolation levels](acid.md).
