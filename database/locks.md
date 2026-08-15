# Locks (shared/exclusive/MVCC)

Mecanismos de control de concurrencia: cómo la base evita que transacciones simultáneas se pisen los datos.

## Race condition — el problema de fondo

Ocurre cuando dos o más operaciones acceden al mismo dato compartido al mismo tiempo, y el resultado final depende del **orden/timing** exacto en que se intercalan — sin ninguna garantía de cuál va a ganar. El caso clásico es el *lost update*: dos requests leen el mismo valor, cada uno calcula su cambio sobre esa lectura, y el que escribe segundo **pisa** el cambio del primero sin enterarse de que existió.

```sql
-- T1 y T2 corren "al mismo tiempo", ambos arrancan leyendo el mismo stock
-- T1: SELECT stock FROM products WHERE id = 1;  → lee 5
-- T2: SELECT stock FROM products WHERE id = 1;  → lee 5 (todavía no vio el cambio de T1)

-- T1: UPDATE products SET stock = 4 WHERE id = 1;  -- 5 - 1, basado en lo que leyó
-- T2: UPDATE products SET stock = 4 WHERE id = 1;  -- 5 - 1, basado en lo que leyó

-- resultado: stock = 4, pero deberían haberse vendido 2 unidades → stock real = 3
-- el UPDATE de T2 pisó el de T1 sin que ninguno de los dos se enterara del otro
```

## Sección crítica — el concepto general detrás de la solución

El fragmento de código que toca el dato compartido (el `SELECT` + `UPDATE` de arriba) es una **sección crítica**: cualquier tramo que no puede ejecutarse por más de un hilo/proceso a la vez sin arriesgar una race condition. Las primitivas clásicas para protegerla son el **mutex** (exclusión mutua — un solo acceso a la vez) y el **semáforo** (permite hasta N accesos simultáneos; un mutex es, en el fondo, un semáforo con N=1).

```python
import threading

lock = threading.Lock()  # mutex: exclusión mutua, un solo hilo a la vez

def decrement_stock():
    with lock:  # sección crítica: nadie más entra hasta que este bloque termine
        stock[product_id] -= 1
```

Los locks de este archivo (pessimistic/optimistic, shared/exclusive, más abajo) son **un caso particular** de este concepto — la implementación de "proteger una sección crítica" a nivel de base de datos. No es la única: un [distributed lock con Redis](redis.md#no-es-solo-cache) protege la misma idea, pero coordinando entre procesos/instancias en vez de entre transacciones de una DB; y el `threading.Lock()` de arriba la protege dentro de un único proceso, sin ninguna base de datos de por medio. Todo lo que sigue en este archivo son formas de resolver esto específicamente para datos en una DB: o se bloquea el acceso concurrente (pessimistic locking, shared/exclusive locks), o se detecta el conflicto al momento de escribir y se rechaza la escritura pisada (optimistic locking).

## Pessimistic vs Optimistic locking

- **Pessimistic**: asumo que va a haber conflicto, así que bloqueo el recurso antes de tocarlo (`SELECT ... FOR UPDATE`). Otros procesos esperan.
- **Optimistic**: asumo que no va a haber conflicto, dejo que todos lean/escriban libremente, y verifico al momento de escribir (ej. columna `version`) si alguien más modificó el dato mientras tanto — si es así, rechazo el update.

```sql
-- Optimistic locking con columna version
UPDATE products SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 5;
-- Si affected rows = 0 → alguien más lo modificó, hay que reintentar
```

## Shared lock (S) vs Exclusive lock (X)

| Lock | Permite a otros leer | Permite a otros escribir | Uso |
|---|---|---|---|
| **Shared (S)** | Sí (otro S) | No | `SELECT ... FOR SHARE` |
| **Exclusive (X)** | No | No | `UPDATE`, `DELETE`, `SELECT ... FOR UPDATE` |

```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- lock exclusivo de la fila
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT; -- libera el lock
```

## Row-level vs table-level

- La mayoría de los motores modernos (Postgres, MySQL InnoDB) lockean a **nivel fila** por defecto — mucho mejor para concurrencia que lockear la tabla entera.
- `LOCK TABLE` existe para casos puntuales (ej. migraciones, operaciones batch) pero bloquea a todos los demás.

## Deadlocks

Dos transacciones se bloquean mutuamente esperando el lock que tiene la otra.

```
T1: lock fila A, espera fila B
T2: lock fila B, espera fila A
→ deadlock
```

El motor detecta el ciclo y aborta una de las dos transacciones (la víctima recibe un error y debe reintentar). **Mitigación**: siempre lockear recursos en el mismo orden en toda la aplicación (ej. siempre por `id` ascendente).

## MVCC (Multi-Version Concurrency Control)

En vez de bloquear lecturas, Postgres y MySQL InnoDB mantienen **múltiples versiones** de cada fila:

- Cada transacción ve un *snapshot* consistente de los datos según su isolation level, sin bloquear a los que escriben.
- Un `UPDATE` no sobreescribe la fila en el lugar: crea una nueva versión y marca la vieja como obsoleta (Postgres) o la mueve al *undo log* (MySQL InnoDB).
- Un proceso de limpieza (`VACUUM` en Postgres) elimina versiones viejas que ya nadie necesita.

**Consecuencia práctica**: con MVCC, lectores nunca bloquean escritores ni viceversa (`SELECT` no espera a un `UPDATE` en curso) — solo escritor vs escritor genera contención real.

Ver también [ACID / transacciones / isolation levels](acid-transacciones-isolation.md), que depende directamente de estos mecanismos.
