# ACID / Transactions / Isolation Levels

## ACID

- **Atomicity**: the transaction applies completely or not at all (`BEGIN`/`COMMIT`/`ROLLBACK`).
- **Consistency**: the transaction takes the database from one valid state to another valid state (respects constraints, foreign keys, triggers).
- **Isolation**: concurrent transactions don't step on each other — each sees a coherent state, controlled by the **isolation level**.
- **Durability**: once `COMMIT` runs, the change survives a crash (persisted to disk / WAL).

## Basic transactions

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- or ROLLBACK if something fails
```

If the process dies between the two `UPDATE`s without a `COMMIT`, on reconnecting the transaction doesn't exist: neither change was applied (atomicity).

## The three "phenomena" that define isolation levels

| Phenomenon | What happens |
|---|---|
| **Dirty read** | Reading data another transaction wrote but hasn't committed yet (and might roll back). |
| **Non-repeatable read** | Reading the same row twice in the same transaction and getting different values because another transaction modified and committed it in between. |
| **Phantom read** | Repeating the same query with a `WHERE` and rows appear/disappear because another transaction inserted/deleted rows matching the filter. |

## Isolation levels (SQL standard)

| Level | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible* |
| Serializable | Prevented | Prevented | Prevented |

\* In Postgres, `Repeatable Read` uses an MVCC snapshot and in practice also prevents phantom reads (stricter than the SQL standard).

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- ...
COMMIT;
```

## Default per engine

- **Postgres**: `Read Committed` by default.
- **MySQL (InnoDB)**: `Repeatable Read` by default.
- **SQL Server**: `Read Committed` by default.

A higher isolation level → more safety, but more contention (locks) and more risk of *serialization failure* errors that require retrying the transaction. It's a consistency-vs-throughput trade-off.

See also [Locks (shared/exclusive/MVCC)](locks.md) — the internal mechanism that enforces these levels — and [Rollback / savepoints](rollback-savepoints.md).
