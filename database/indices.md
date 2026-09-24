# Índices

Estructura de datos auxiliar que permite encontrar filas sin recorrer toda la tabla (evita el *full table scan*).

## Por qué existen

Sin índice, `WHERE email = 'x@mail.com'` es O(n): recorre todas las filas. Con un índice tipo B-tree sobre `email`, la búsqueda es O(log n).

```sql
CREATE INDEX idx_users_email ON users(email);
```

## Tipos más comunes

| Tipo | Uso típico | Motor |
|---|---|---|
| **B-tree** | Default. Igualdad, rangos (`<`, `>`, `BETWEEN`), `ORDER BY` | Todos |
| **Hash** | Solo igualdad (`=`), no sirve para rangos | Postgres, MySQL |
| **GIN** | Full-text search, arrays, JSONB | Postgres |
| **GiST** | Datos geométricos, rangos, búsquedas de proximidad | Postgres |
| **Bitmap** | Columnas de baja cardinalidad (pocos valores distintos) combinadas | Oracle, data warehouses |

## Índices compuestos

```sql
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

- El orden de las columnas importa: este índice sirve para `WHERE user_id = ?` y para `WHERE user_id = ? AND created_at > ?`, pero **no** sirve eficientemente para `WHERE created_at > ?` solo (regla del *leftmost prefix*).

## Covering index

Un índice que incluye todas las columnas que la query necesita, permitiendo responder sin ir a la tabla (*index-only scan*):

```sql
CREATE INDEX idx_orders_covering ON orders(user_id) INCLUDE (status, total);
```

## Clustered vs non-clustered

- **Clustered**: los datos de la tabla se almacenan físicamente en el orden del índice (ej. la PK en SQL Server/MySQL InnoDB). Solo puede haber uno por tabla.
- **Non-clustered**: estructura separada que apunta a la ubicación de la fila. Puede haber varios.
- Postgres no tiene clustered index real (usa *heap* + índices que apuntan al heap; `CLUSTER` reordena físicamente una sola vez, no lo mantiene).

## Costo: no son gratis

- Cada índice acelera lecturas pero **ralentiza escrituras** (`INSERT`/`UPDATE`/`DELETE` deben mantener también el índice actualizado).
- Ocupan espacio en disco.
- Indexar las columnas usadas en `JOIN` (típicamente las foreign keys) acelera esa operación por el mismo motivo que acelera un `WHERE`: el motor busca por índice en vez de escanear la tabla completa del otro lado del join.

**"Indexar todo por las dudas" no es la solución** — es el error contrario al de no indexar nada. Cada índice de más:
- Suma su propio costo de escritura, y ese costo se **acumula**: una tabla con 5 índices paga el costo de mantenimiento de los 5 en cada `INSERT`, no del más caro.
- Puede quedar **redundante**: un índice sobre `(user_id)` es innecesario si ya existe uno sobre `(user_id, created_at)` — el compuesto ya cubre las queries que solo filtran por `user_id` (ver [índices compuestos](#índices-compuestos) y la regla de *leftmost prefix*).
- Sirve para nada si la columna tiene **baja cardinalidad** (ej. un `boolean`, o un `status` con 3 valores posibles) — el motor puede decidir que escanear la tabla completa es más barato que usar ese índice, salvo con un [índice parcial](#índice-parcial-postgres) sobre el subconjunto realmente selectivo.

La regla práctica es indexar según **los patrones de query reales** (mirando qué se usa en `WHERE`/`JOIN`/`ORDER BY` en producción, no "por si acaso"), y confirmar con [`EXPLAIN ANALYZE`](#cómo-saber-si-se-está-usando) que el índice efectivamente se usa antes de darlo por bueno.

## Índice parcial (Postgres)

```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

Útil cuando solo un subconjunto de filas se consulta frecuentemente.

## Cómo saber si se está usando

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'x@mail.com';
```

Buscar `Index Scan` / `Index Only Scan` en el plan (vs `Seq Scan`). Ver [Diagnóstico Base de Datos](../diagnostics/database.es.md) para el detalle de `EXPLAIN ANALYZE`.

Relacionado: [Queries non-sargable](queries-non-sargable.md) — patrones de queries que impiden que el motor use el índice aunque exista.
