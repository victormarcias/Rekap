# Database Migrations

How a database's schema evolves over time without breaking production — the challenge isn't writing the `ALTER TABLE`, it's coordinating it with the app code that's already running.

## 1. Schema versioning

Every schema change (adding a column, creating a table, changing a type) is captured in a versioned, ordered migration file (numbering or timestamp), always applied in sequence. A special table (e.g. `schema_migrations`) tracks which migrations have already run in each environment — without that record there's no reliable way to know whether production's schema is up to date with the code, or to apply exactly what's missing when spinning up a new environment.

```sql
-- 001_create_users_table.sql
CREATE TABLE users (id SERIAL PRIMARY KEY, email TEXT NOT NULL);

-- 002_add_active_column.sql
ALTER TABLE users ADD COLUMN active BOOLEAN DEFAULT true;

-- the migration tool itself tracks which versions have already run
SELECT * FROM schema_migrations;
-- version | applied_at
-- 001     | 2026-01-10
-- 002     | 2026-02-03
```

## 2. Forward-only vs reversible

A **reversible** migration defines an `up` step (apply) and a `down` step (undo), meant to let you roll back if something goes wrong. A **forward-only** migration has no `down`: if something goes wrong, a new migration is written to fix the problem, you never "go back."

```sql
-- 002_add_active_column.up.sql
ALTER TABLE users ADD COLUMN active BOOLEAN DEFAULT true;

-- 002_add_active_column.down.sql
ALTER TABLE users DROP COLUMN active;
-- ⚠️ if real data was already written to that column between the up and the down, it's lost on rollback
```

In practice, many teams end up going forward-only even when the tool supports `down`: the `down` is almost never tested (nobody runs it until the day they desperately need it), and by then the data has already changed in ways that make the original `down` invalid — reverting a schema after real data has been written is inherently riskier than reverting application code.

## 3. How it's coordinated with deploys

The problem isn't the migration itself, it's the **order** relative to the code deploy. With more than one instance of the app running (a rolling deploy, the normal case for any system with more than one server), old and new code coexist during the rollout window. A migration that isn't compatible with the old code (e.g. renaming a column) breaks the old instances still serving traffic while the new code finishes rolling out.

```sql
-- ❌ a direct rename, deployed alongside the new code:
-- for the seconds/minutes the old code is still running,
-- every request that touches this column fails because "email" no longer exists
ALTER TABLE users RENAME COLUMN email TO email_address;
```

The fix isn't "do it faster" — it's splitting the change into steps that are compatible with both versions of the code at the same time. That's the **expand/contract pattern**.

## 4. Expand/contract pattern (zero-downtime)

Instead of one destructive change all at once, it's split into phases where the schema is always valid for both the old code and the new:

```sql
-- Phase 1 — Expand: add the new column without touching the old one.
-- The old code doesn't even know it exists, keeps working the same.
ALTER TABLE users ADD COLUMN email_address TEXT;

-- Phase 2 — Backfill + dual write: copy existing data into the new column,
-- while the app code (already deployed) writes to both columns at once.
UPDATE users SET email_address = email WHERE email_address IS NULL;

-- Phase 3 — Deploy the code that only reads/writes email_address,
-- only once the backfill is confirmed done and no old instances are running.

-- Phase 4 — Contract: drop the old column, in a separate, later migration,
-- only after confirming nothing still uses it.
ALTER TABLE users DROP COLUMN email;
```

Each phase, on its own, is compatible with both the old and new code — there's never a moment where one version of the app breaks against the current schema. The cost is obvious: 4 steps (and 4 deploys/migrations) instead of one, but it's the price of no downtime.

## Note on locks during the migration

An `ALTER TABLE` isn't free on large tables: adding a column with a default value in old versions of Postgres (<11) rewrote the entire table under an exclusive lock, blocking reads and writes for the whole operation. See [Locks](locks.md) — it's worth confirming what kind of lock each operation takes on your specific engine before running a migration on a table with real production traffic.

---
Related: [Locks](locks.md), [ACID / transactions / isolation levels](acid.md), [System quality attributes](../system-design/atributos-de-calidad.md) (Availability).
