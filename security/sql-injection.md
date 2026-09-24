# SQL Injection

Inserting SQL code through an input that gets concatenated directly into a query, instead of being treated as data — the database engine has no way to tell "this is part of the instruction" apart from "this is a value" if both arrive mixed together in the same string.

```python
# ❌ vulnerable: the user's input is pasted directly into the SQL
user_id = request.args["id"]
query = f"SELECT * FROM users WHERE id = {user_id}"

# if user_id = "1 OR 1=1"        → the condition is always true, returns ALL users
# if user_id = "1; DROP TABLE users; --"  → drops the entire table (if the driver allows multi-statement)

# ✅ safe: parametrized query — the driver sends the SQL and the data separately,
# they're never concatenated as text
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

## The mistake of thinking an ORM protects you on its own

An ORM parametrizes automatically when you use its API — but if you build SQL by hand within the same project (with f-strings, `.format()`, or concatenation), the vulnerability is still there, ORM or not:

```python
# ❌ still vulnerable — the ORM isn't involved, it's a hand-built string
session.execute(f"SELECT * FROM users WHERE email = '{email}'")

# ✅ the ORM parametrizes because you're using its API, not your own string
session.query(User).filter(User.email == email)
```

## How it's prevented

- **Parametrized queries / prepared statements**: always — never interpolate a user value directly into SQL, not even "just for one specific case."
- **An ORM used as intended**: avoids the problem by design, as long as the abstraction isn't broken with hand-built raw SQL.
- **Least privilege in the DB**: the application's connection user shouldn't have `DROP`/`ALTER` permission if the app never needs it in production — limits the damage even if something slips through.

---
Related: [SQL Engines](../database/sql-engines.md), [Controller / Service / Repository](../backend/controller-service-repository.md#orm-object-relational-mapping), [SSRF](ssrf.md).
