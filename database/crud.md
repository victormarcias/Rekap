# CRUD

The four basic persistence operations: Create, Read, Update, Delete.

## Create — INSERT

```sql
-- Single row
INSERT INTO users (name, email) VALUES ('Ana', 'ana@mail.com');

-- Multi-row (a single statement, more efficient than N inserts)
INSERT INTO users (name, email) VALUES
  ('Ana', 'ana@mail.com'),
  ('Beto', 'beto@mail.com');

-- INSERT ... SELECT (copy/transform from another table)
INSERT INTO users_archive (name, email)
SELECT name, email FROM users WHERE created_at < '2020-01-01';

-- RETURNING (Postgres) — avoids an extra SELECT to get the generated id
INSERT INTO users (name) VALUES ('Ana') RETURNING id;
```

## Read — SELECT

```sql
SELECT id, name FROM users
WHERE active = true
ORDER BY created_at DESC
LIMIT 20 OFFSET 40; -- pagination
```

- `WHERE` filters rows, `HAVING` filters groups (post `GROUP BY`).
- Careful with `SELECT *` in production: it couples the consumer to the full schema and pulls in unnecessary columns.
- `LIMIT/OFFSET` for pagination is simple but O(n) at large offsets — for efficient pagination use **keyset pagination** (`WHERE id > :last_id ORDER BY id LIMIT 20`).

## Update — UPDATE

```sql
UPDATE users SET active = false WHERE last_login < now() - interval '1 year';
```

- **Never** run an `UPDATE`/`DELETE` with no `WHERE` in production without confirming first — it affects every row.
- Prefer transactions (`BEGIN ... COMMIT`) so you can `ROLLBACK` if the result isn't what you expected. See [ACID / transactions / isolation levels](acid.md).

## Delete — DELETE vs TRUNCATE vs DROP

| Command | What it deletes | Reversible (with a transaction) | Resets auto-increment | Fires triggers |
|---|---|---|---|---|
| `DELETE FROM t WHERE ...` | Specific rows | ✅ | ❌ | ✅ |
| `TRUNCATE TABLE t` | All rows | Depends on the engine (Postgres: yes, within a transaction) | ✅ | No (generally) |
| `DROP TABLE t` | The entire table (structure + data) | Depends on the engine | N/A | N/A |

## Referential actions — what happens to related rows on delete

When a table has a foreign key pointing to another (e.g. `posts.user_id` → `users.id`), you have to decide what happens to those dependent rows if the referenced row is deleted. This is defined when creating the foreign key:

```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id) ON DELETE CASCADE
);
```

- **`ON DELETE CASCADE`**: if the user is deleted, every row that references it (their posts, their comments) gets deleted automatically. Convenient, but dangerous if unexpected — a single `DELETE` can cascade across several tables.
- **`ON DELETE RESTRICT`** (or the default behavior in many engines if nothing is specified): won't let you delete the user while it still has associated posts or comments — dependencies have to be deleted first, by hand or in an explicit flow.
- **`ON DELETE SET NULL`**: deletes the user, but instead of deleting the posts/comments, sets their `user_id = NULL` (e.g. to show "deleted user" instead of losing the content). Requires the `user_id` column to allow `NULL`.

**Which one to use**: `CASCADE` when the child doesn't make sense without the parent (comments on a post that gets deleted). `RESTRICT` when accidentally deleting something with dependencies is more dangerous than being annoying (deleting a user with active orders). `SET NULL` when the dependent content should survive even if it loses the reference (posts from a deleted user that stay public).

## Upsert (Create or Update depending on existence)

```sql
-- Postgres / SQLite
INSERT INTO users (id, name) VALUES (1, 'Ana')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;

-- MySQL
INSERT INTO users (id, name) VALUES (1, 'Ana')
ON DUPLICATE KEY UPDATE name = VALUES(name);
```

## iOS Analogy

`INSERT`/`UPDATE`/`DELETE` + `SELECT` is to the backend what `Core Data` (`NSManagedObjectContext.save()`, fetch requests) is to the local world: the difference is that here there's no single in-memory "context" — every query hits the database directly (or via a connection pool), so concurrency control and transactions are explicit.
