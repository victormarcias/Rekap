# RDBMS (Relational Database Management System)

## Qué es

Software que organiza datos en **tablas** (filas y columnas) relacionadas entre sí mediante claves (foreign keys), con **SQL** como lenguaje de consulta y garantías **ACID** para las transacciones. Implementa el modelo relacional (teoría de Edgar Codd, 1970). Ejemplos: PostgreSQL, MySQL, SQL Server, Oracle — cada uno con su propio motor por debajo, pero el mismo modelo de datos arriba.

## Cuándo usarlo (y cuándo no)

**Usarlo cuando:**
- Los datos tienen relaciones claras entre entidades y la integridad importa (un pedido no puede existir sin un cliente válido — eso lo garantiza una foreign key, no el código de la aplicación).
- Las transacciones tienen que ser atómicas y consistentes (plata, inventario, reservas — todo o nada, nunca un estado a medias).
- Las queries necesitan joins y agregaciones complejas sobre datos estructurados.
- El schema es relativamente estable — no cambia de forma todos los días.

**No usarlo (o no solo) cuando:**
- El patrón de acceso es simple y conocido de antemano (buscar por id, sin joins) y la prioridad es escalar horizontalmente sin límite — ahí conviene [NoSQL](nosql.md).
- El schema cambia todo el tiempo y forzar una estructura fija cuesta más de lo que aporta.

## Cómo funciona por dentro

Una query SQL atraviesa varias capas antes de devolver un resultado:

```
Cliente (SQL) → Parser → Query Optimizer → Execution Engine → Storage Engine → Disco
                              ↑                    ↓
                        elige el plan      usa Índices para no
                        más barato         escanear todo
```

1. **Parser**: valida que el SQL sea sintácticamente correcto.
2. **Query Optimizer**: entre varias formas posibles de ejecutar la query, elige la que estima más barata — usando estadísticas de las tablas (ver [Estadísticas del planner](../diagnostics/database.es.md#estadísticas-del-planner)) y decidiendo si conviene un [índice](indices.md) o escanear la tabla entera.
3. **Execution Engine**: corre el plan elegido, leyendo/escribiendo filas.
4. **Storage Engine**: no lee/escribe fila por fila directo a disco — trabaja en **páginas** (bloques de varios KB), y mantiene un **buffer pool** en memoria con las páginas usadas más recientemente, para no ir a disco en cada operación (el mismo principio que un cache — ver [Diagnóstico Backend](../diagnostics/backend.es.md#falta-de-cache) para el concepto de cache en general).

## Write-Ahead Log (WAL) — cómo no se pierde nada si el servidor se cae

Antes de modificar los datos reales en disco, el cambio se escribe primero a un **log secuencial** (el WAL). Recién después de que ese log confirma el cambio, se aplica a las páginas de datos reales — que pueden quedar en memoria (buffer pool) un rato antes de sincronizarse a disco, porque escribir a disco en cada transacción sería carísimo.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT; -- acá el WAL ya confirmó el cambio; la página de datos puede sincronizarse a disco después
```

Si el servidor se cae entre el `COMMIT` y que la página llegue a disco, al reiniciar el motor **reproduce el WAL** desde el último punto seguro — recupera todo lo que estaba confirmado, sin haber tenido que sincronizar cada página en cada transacción. Es el mecanismo concreto detrás de la **D** (Durability) de [ACID](acid-transacciones-isolation.md#acid): la garantía no viene de escribir todo a disco siempre, viene de este log que sí se escribe de forma segura y secuencial (mucho más barato que escrituras aleatorias por todo el archivo de datos).

---
Relacionado: [ACID / transacciones / isolation levels](acid-transacciones-isolation.md), [Índices](indices.md), [Query Optimization](query-optimization.md), [Locks](locks.md), [NoSQL](nosql.md).
