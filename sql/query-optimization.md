# Query Optimization

Checklist de lo que revisar antes de dar una query por buena en producción.

## Joins correctos — evitar productos cartesianos

Un `JOIN` sin condición (o con una condición incompleta/incorrecta) no relaciona las tablas — las **multiplica**: cada fila de un lado se combina con cada fila que matchea del otro, silenciosamente, sin ningún error. El resultado es muchas más filas de las esperadas, a veces órdenes de magnitud más.

```sql
-- ❌ sin condición ON: cada fila de orders se combina con CADA fila de order_items
SELECT * FROM orders, order_items;
-- con 1.000 orders y 5.000 items, esto devuelve 5.000.000 de filas, no 5.000

-- ❌ condición de join por una columna que no identifica la relación real (fan-out)
SELECT * FROM orders o
JOIN order_items oi ON o.user_id = oi.user_id  -- debería ser o.id = oi.order_id
-- cada order de un usuario se combina con TODOS los items de TODAS sus orders, no solo la suya

-- ✅ condición de join correcta, por la clave que realmente relaciona las filas
SELECT * FROM orders o
JOIN order_items oi ON o.id = oi.order_id;
```

**Cómo detectarlo**: el resultado tiene muchas más filas de las que el negocio esperaría para ese filtro, o `EXPLAIN ANALYZE` muestra una estimación de filas absurdamente alta en algún paso del plan — ver [Diagnóstico Base de Datos](../diagnostico/base-de-datos.md#execution-plan).

## Subqueries — cuándo evitarlas

El problema no son las subqueries en general, es la **subquery correlacionada**: una que referencia una columna de la query externa, y por eso el motor la re-ejecuta **una vez por cada fila** del resultado externo — básicamente el mismo problema de fondo que [N+1](../diagnostico/backend.md#problema-n1), pero dentro de una sola sentencia SQL en vez de entre requests.

```sql
-- ❌ subquery correlacionada: corre una vez POR CADA fila de orders
SELECT o.id, o.total,
  (SELECT COUNT(*) FROM order_items oi WHERE oi.order_id = o.id) AS item_count
FROM orders o;

-- ✅ JOIN + GROUP BY: el planner lo resuelve en un solo pase (hash join o merge join)
SELECT o.id, o.total, COUNT(oi.id) AS item_count
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.id
GROUP BY o.id, o.total;
```

No toda subquery es mala — una subquery en `WHERE ... EXISTS (...)` para chequear existencia puede ser tan buena o mejor que un `JOIN`, porque el planner puede cortar apenas encuentra la primera coincidencia sin traer filas de más. La regla real es evitar las **correlacionadas ejecutadas por fila** y el anidamiento profundo (subquery dentro de subquery dentro de subquery), no las subqueries como categoría.

## El resto del checklist — ya cubierto en otros machetes

- **Traer solo las columnas necesarias** (no `SELECT *`) → [CRUD](crud.md#read--select)
- **Filtrar correctamente con `WHERE` vs `HAVING`** (filas vs grupos) → [CRUD](crud.md#read--select)
- **Evitar `LIKE` innecesario** (sobre todo con wildcard al inicio, que rompe el uso del índice) → [Queries non-sargable](queries-non-sargable.md)

---
Relacionado: [Índices](indices.md), [Diagnóstico Base de Datos](../diagnostico/base-de-datos.md), [N+1](../diagnostico/backend.md#problema-n1).
