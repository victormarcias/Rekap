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

### Dirty read

```sql
-- 1. T1: BEGIN; UPDATE accounts SET balance = 0 WHERE id = 1;     -- not committed yet
-- 2. T2: SELECT balance FROM accounts WHERE id = 1;               -- returns 0  ← dirty read
-- 3. T1: ROLLBACK;                                                -- that 0 never existed
```

T2 acted on a value that was never real — e.g. it approved a purchase against a balance of 0 that got rolled back.

### Non-repeatable read

```sql
-- 1. T1: BEGIN; SELECT balance FROM accounts WHERE id = 1;        -- 100
-- 2. T2: UPDATE accounts SET balance = 50 WHERE id = 1; COMMIT;
-- 3. T1: SELECT balance FROM accounts WHERE id = 1;               -- 50  ← same row, same transaction, different value
```

The data T1 read is **a row that changed** — a report that reads the same balance twice in one transaction can end up with numbers that don't add up.

### Phantom read

```sql
-- 1. T1: BEGIN; SELECT count(*) FROM orders WHERE status = 'pending';   -- 3
-- 2. T2: INSERT INTO orders (status) VALUES ('pending'); COMMIT;
-- 3. T1: SELECT count(*) FROM orders WHERE status = 'pending';          -- 4  ← a new row (the "phantom") appeared
```

The difference with the previous one: here no row T1 already read was modified — **new rows** matching the `WHERE` appeared (or disappeared). Locking only the rows already read doesn't prevent it, because the phantom row didn't exist yet to be locked.
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

## How each level prevents them

Two strategies, depending on the engine (see [Locks](locks.md)):

- **Locks (pessimistic)**: reads take [shared locks](locks.md#shared-lock-s-vs-exclusive-lock-x) that block writers. SQL Server works this way by default.
- **MVCC (snapshots)**: reads see a consistent version of the data and block nobody — see [MVCC](locks.md#mvcc-multi-version-concurrency-control). Postgres and MySQL InnoDB work this way.

| Level | Lock-based (SQL Server) | MVCC (Postgres) |
|---|---|---|
| Read Uncommitted | Reads take no locks → dirty reads possible | Not implemented: behaves as Read Committed |
| Read Committed | Shared lock released right after reading → no dirty reads, but a second read can differ | Each **statement** sees a snapshot taken when it starts |
| Repeatable Read | Shared locks held until the end of the transaction → the rows already read can't change. New rows can still be inserted (phantoms) | One snapshot for the **whole transaction** → also no phantoms. If it tries to modify a row another transaction already changed → serialization failure |
| Serializable | Range locks on everything the query scanned → nobody can insert into that range | SSI (Serializable Snapshot Isolation): snapshots plus detection of conflicting patterns; one of the transactions gets aborted |

MySQL InnoDB, at `Repeatable Read`, uses a snapshot for plain `SELECT`s and *next-key locks* (row + the gap before it) for `SELECT ... FOR UPDATE`, which is what stops other transactions from inserting phantoms into that gap.

The price of the higher levels: a **serialization failure** is not a bug, it's the database saying "this transaction conflicted with another one — run it again". The application has to retry the whole transaction:

```python
import psycopg
from psycopg.errors import SerializationFailure

def transfer(conn, src, dst, amount, retries=3):
    for _ in range(retries):
        try:
            with conn.transaction():
                conn.execute("SET TRANSACTION ISOLATION LEVEL SERIALIZABLE")
                conn.execute("UPDATE accounts SET balance = balance - %s WHERE id = %s", (amount, src))
                conn.execute("UPDATE accounts SET balance = balance + %s WHERE id = %s", (amount, dst))
            return
        except SerializationFailure:
            continue  # another transaction conflicted — retrying the whole transaction is safe
    raise RuntimeError("transfer failed after retries")
```

## Default per engine

- **Postgres**: `Read Committed` by default.
- **MySQL (InnoDB)**: `Repeatable Read` by default.
- **SQL Server**: `Read Committed` by default.

A higher isolation level → more safety, but more contention (locks) and more risk of *serialization failure* errors that require retrying the transaction. It's a consistency-vs-throughput trade-off.

## Managed databases on AWS and GCP

A managed service doesn't change the semantics: **RDS and Cloud SQL run the same engine**, so the same defaults and the same phenomena apply. What changes is where the default gets configured (RDS *parameter group*, Cloud SQL *database flags*) — or just set it from SQL, which works on any of them:

```sql
ALTER DATABASE mydb SET default_transaction_isolation = 'repeatable read';  -- Postgres: RDS, Aurora, Cloud SQL, AlloyDB
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;                               -- or per transaction, right after BEGIN
```

| Service | Default isolation | Notes |
|---|---|---|
| **AWS RDS / Aurora PostgreSQL** | Read Committed | Same as Postgres |
| **AWS RDS / Aurora MySQL** | Repeatable Read | Same as InnoDB |
| **AWS RDS SQL Server** | Read Committed | Lock-based by default |
| **AWS DynamoDB** | Single reads: no transaction involved (eventually consistent unless strongly consistent is requested) | `TransactWriteItems` / `TransactGetItems` are **serializable** across up to 100 items |
| **GCP Cloud SQL** (PostgreSQL / MySQL / SQL Server) | The engine's default | Same engine, same behavior |
| **GCP AlloyDB** | Read Committed | PostgreSQL-compatible |
| **GCP Cloud Spanner** | Serializable (strict, with external consistency) | Read-write transactions are always at the strictest level, even across regions |
| **GCP Firestore** | Serializable | Transactions retry automatically on contention |
| **GCP BigQuery** | Snapshot isolation | In multi-statement transactions; not meant for OLTP |

Reading from a **read replica** (RDS, Aurora, Cloud SQL) is a separate matter from isolation: the replica can lag behind the primary, so a read there can return older data than a transaction that already committed on the primary.

See also [Locks (shared/exclusive/MVCC)](locks.md) — the internal mechanism that enforces these levels — and [Rollback / savepoints](rollback-savepoints.md).
