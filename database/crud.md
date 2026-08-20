# CRUD

Las cuatro operaciones básicas de persistencia: Create, Read, Update, Delete.

## Create — INSERT

```sql
-- Single row
INSERT INTO users (name, email) VALUES ('Ana', 'ana@mail.com');

-- Multi-row (una sola sentencia, más eficiente que N inserts)
INSERT INTO users (name, email) VALUES
  ('Ana', 'ana@mail.com'),
  ('Beto', 'beto@mail.com');

-- INSERT ... SELECT (copiar/transformar desde otra tabla)
INSERT INTO users_archive (name, email)
SELECT name, email FROM users WHERE created_at < '2020-01-01';

-- RETURNING (Postgres) — evita un SELECT extra para obtener el id generado
INSERT INTO users (name) VALUES ('Ana') RETURNING id;
```

## Read — SELECT

```sql
SELECT id, name FROM users
WHERE active = true
ORDER BY created_at DESC
LIMIT 20 OFFSET 40; -- paginación
```

- `WHERE` filtra filas, `HAVING` filtra grupos (post `GROUP BY`).
- Cuidado con `SELECT *` en producción: acopla el consumidor al schema completo y trae columnas innecesarias.
- `LIMIT/OFFSET` para paginar es simple pero O(n) en offsets grandes — para paginación eficiente usar **keyset pagination** (`WHERE id > :last_id ORDER BY id LIMIT 20`).

## Update — UPDATE

```sql
UPDATE users SET active = false WHERE last_login < now() - interval '1 year';
```

- **Nunca** correr un `UPDATE`/`DELETE` sin `WHERE` en producción sin confirmar antes — afecta todas las filas.
- Preferir transacciones (`BEGIN ... COMMIT`) para poder hacer `ROLLBACK` si el resultado no es el esperado. Ver [ACID / transacciones / isolation levels](acid-transacciones-isolation.md).

## Delete — DELETE vs TRUNCATE vs DROP

| Comando | Qué borra | Reversible (con transacción) | Reinicia auto-increment | Dispara triggers |
|---|---|---|---|---|
| `DELETE FROM t WHERE ...` | Filas específicas | Sí | No | Sí |
| `TRUNCATE TABLE t` | Todas las filas | Depende del motor (Postgres sí dentro de transacción) | Sí | No (generalmente) |
| `DROP TABLE t` | La tabla entera (estructura + datos) | Depende del motor | N/A | N/A |

## Acciones referenciales — qué pasa con las filas relacionadas al borrar

Cuando una tabla tiene una foreign key hacia otra (ej. `posts.user_id` → `users.id`), hay que decidir qué pasa con esas filas dependientes si se borra la fila referenciada. Se define al crear la foreign key:

```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id) ON DELETE CASCADE
);
```

- **`ON DELETE CASCADE`**: si se borra el usuario, se borran automáticamente todas las filas que lo referencian (sus posts, sus comentarios). Cómodo, pero peligroso si no se lo espera — un solo `DELETE` puede propagarse en cadena por varias tablas.
- **`ON DELETE RESTRICT`** (o el comportamiento por default en muchos motores si no se especifica nada): no deja borrar al usuario mientras todavía tenga posts o comentarios asociados — hay que borrar las dependencias primero, a mano o en un flujo explícito.
- **`ON DELETE SET NULL`**: borra al usuario, pero en vez de borrar los posts/comentarios, les pone `user_id = NULL` (ej. para mostrar "usuario eliminado" en vez de perder el contenido). Requiere que la columna `user_id` permita `NULL`.

**Cuál usar**: `CASCADE` cuando el hijo no tiene sentido sin el padre (comentarios de un post que se borra). `RESTRICT` cuando borrar por accidente algo con dependencias es más peligroso que ser molesto (borrar un usuario con pedidos activos). `SET NULL` cuando el contenido dependiente debe sobrevivir aunque pierda la referencia (posts de un usuario eliminado que siguen siendo públicos).

## Upsert (Create o Update según exista)

```sql
-- Postgres / SQLite
INSERT INTO users (id, name) VALUES (1, 'Ana')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;

-- MySQL
INSERT INTO users (id, name) VALUES (1, 'Ana')
ON DUPLICATE KEY UPDATE name = VALUES(name);
```

## Analogía iOS

`INSERT`/`UPDATE`/`DELETE` + `SELECT` es al backend lo que `Core Data` (`NSManagedObjectContext.save()`, fetch requests) es al mundo local: la diferencia es que acá no hay un solo "contexto" en memoria, cada query golpea la base directamente (o vía un connection pool), así que el control de concurrencia y las transacciones son explícitos.
