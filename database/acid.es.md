# ACID / Transacciones / Isolation Levels

## ACID

- **Atomicity**: la transacción se aplica completa o no se aplica nada (`BEGIN`/`COMMIT`/`ROLLBACK`).
- **Consistency**: la transacción lleva la base de un estado válido a otro estado válido (respeta constraints, foreign keys, triggers).
- **Isolation**: transacciones concurrentes no se pisan entre sí — cada una ve un estado coherente, controlado por el **isolation level**.
- **Durability**: una vez hecho `COMMIT`, el cambio sobrevive a un crash (persistido en disco / WAL).

## Transacciones básicas

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- o ROLLBACK si algo falla
```

Si el proceso muere entre los dos `UPDATE` sin `COMMIT`, al reconectar la transacción no existe: ninguno de los dos cambios se aplicó (atomicidad).

## Los tres "phenomena" que definen los isolation levels

| Fenómeno | Qué pasa |
|---|---|
| **Dirty read** | Leo un dato que otra transacción escribió pero todavía no hizo commit (y puede hacer rollback). |
| **Non-repeatable read** | Leo la misma fila dos veces en la misma transacción y obtengo valores distintos porque otra transacción la modificó y comiteó en el medio. |
| **Phantom read** | Repito la misma query con un `WHERE` y aparecen/desaparecen filas porque otra transacción insertó/borró filas que matchean el filtro. |

### Dirty read

```sql
-- 1. T1: BEGIN; UPDATE accounts SET balance = 0 WHERE id = 1;     -- sin commit todavía
-- 2. T2: SELECT balance FROM accounts WHERE id = 1;               -- devuelve 0  ← dirty read
-- 3. T1: ROLLBACK;                                                -- ese 0 nunca existió
```

T2 actuó sobre un valor que nunca fue real — por ejemplo, aprobó una compra contra un saldo de 0 que después se deshizo.

### Non-repeatable read

```sql
-- 1. T1: BEGIN; SELECT balance FROM accounts WHERE id = 1;        -- 100
-- 2. T2: UPDATE accounts SET balance = 50 WHERE id = 1; COMMIT;
-- 3. T1: SELECT balance FROM accounts WHERE id = 1;               -- 50  ← misma fila, misma transacción, otro valor
```

El dato que leyó T1 es **una fila que cambió** — un reporte que lee el mismo saldo dos veces en una transacción puede terminar con números que no cierran.

### Phantom read

```sql
-- 1. T1: BEGIN; SELECT count(*) FROM orders WHERE status = 'pending';   -- 3
-- 2. T2: INSERT INTO orders (status) VALUES ('pending'); COMMIT;
-- 3. T1: SELECT count(*) FROM orders WHERE status = 'pending';          -- 4  ← apareció una fila nueva (el "fantasma")
```

La diferencia con el anterior: acá no se modificó ninguna fila que T1 ya hubiera leído — aparecieron (o desaparecieron) **filas nuevas** que cumplen el `WHERE`. Bloquear solo las filas ya leídas no lo evita, porque la fila fantasma todavía no existía para bloquearla.
## Isolation levels (SQL standard)

| Nivel | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| Read Uncommitted | Posible | Posible | Posible |
| Read Committed | Evitado | Posible | Posible |
| Repeatable Read | Evitado | Evitado | Posible* |
| Serializable | Evitado | Evitado | Evitado |

\* En Postgres, `Repeatable Read` usa snapshot MVCC y en la práctica también evita phantom reads (más estricto que el estándar SQL).

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- ...
COMMIT;
```

## Cómo previene cada nivel

Dos estrategias, según el motor (ver [Locks](locks.es.md)):

- **Locks (pesimista)**: las lecturas toman [shared locks](locks.es.md#shared-lock-s-vs-exclusive-lock-x) que bloquean a los escritores. SQL Server funciona así por defecto.
- **MVCC (snapshots)**: las lecturas ven una versión consistente de los datos y no bloquean a nadie — ver [MVCC](locks.es.md#mvcc-multi-version-concurrency-control). Postgres y MySQL InnoDB funcionan así.

| Nivel | Basado en locks (SQL Server) | MVCC (Postgres) |
|---|---|---|
| Read Uncommitted | Las lecturas no toman locks → dirty reads posibles | No implementado: se comporta como Read Committed |
| Read Committed | Shared lock que se suelta apenas se lee → sin dirty reads, pero una segunda lectura puede diferir | Cada **statement** ve un snapshot tomado al empezar |
| Repeatable Read | Shared locks retenidos hasta el final de la transacción → las filas ya leídas no pueden cambiar. Igual pueden insertarse filas nuevas (phantoms) | Un snapshot para **toda la transacción** → tampoco hay phantoms. Si intenta modificar una fila que otra transacción ya cambió → serialization failure |
| Serializable | Range locks sobre todo lo que escaneó la query → nadie puede insertar en ese rango | SSI (Serializable Snapshot Isolation): snapshots más detección de patrones conflictivos; una de las transacciones se aborta |

MySQL InnoDB, en `Repeatable Read`, usa un snapshot para los `SELECT` comunes y *next-key locks* (la fila + el hueco anterior) para `SELECT ... FOR UPDATE`, que es lo que impide que otras transacciones inserten phantoms en ese hueco.

El precio de los niveles más altos: un **serialization failure** no es un bug, es la base diciendo "esta transacción chocó con otra — correla de nuevo". La aplicación tiene que reintentar la transacción completa:

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
            continue  # otra transacción chocó — reintentar la transacción completa es seguro
    raise RuntimeError("transfer failed after retries")
```

## Default por motor

- **Postgres**: `Read Committed` por defecto.
- **MySQL (InnoDB)**: `Repeatable Read` por defecto.
- **SQL Server**: `Read Committed` por defecto.

A mayor isolation level → mayor seguridad, pero más contención (locks) y más riesgo de errores de tipo *serialization failure* que requieren retry de la transacción. Es un trade-off consistencia vs throughput.

## Bases de datos gestionadas en AWS y GCP

Un servicio gestionado no cambia la semántica: **RDS y Cloud SQL corren el mismo motor**, así que valen los mismos defaults y los mismos fenómenos. Lo que cambia es dónde se configura el default (*parameter group* en RDS, *database flags* en Cloud SQL) — o directamente se setea desde SQL, que funciona en cualquiera:

```sql
ALTER DATABASE mydb SET default_transaction_isolation = 'repeatable read';  -- Postgres: RDS, Aurora, Cloud SQL, AlloyDB
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;                               -- o por transacción, justo después del BEGIN
```

| Servicio | Isolation por defecto | Notas |
|---|---|---|
| **AWS RDS / Aurora PostgreSQL** | Read Committed | Igual que Postgres |
| **AWS RDS / Aurora MySQL** | Repeatable Read | Igual que InnoDB |
| **AWS RDS SQL Server** | Read Committed | Basado en locks por defecto |
| **AWS DynamoDB** | Lecturas simples: sin transacción de por medio (eventually consistent salvo que se pida strongly consistent) | `TransactWriteItems` / `TransactGetItems` son **serializable** sobre hasta 100 ítems |
| **GCP Cloud SQL** (PostgreSQL / MySQL / SQL Server) | El default del motor | Mismo motor, mismo comportamiento |
| **GCP AlloyDB** | Read Committed | Compatible con PostgreSQL |
| **GCP Cloud Spanner** | Serializable (estricto, con external consistency) | Las transacciones de lectura-escritura siempre están en el nivel más estricto, incluso entre regiones |
| **GCP Firestore** | Serializable | Las transacciones se reintentan solas ante contención |
| **GCP BigQuery** | Snapshot isolation | En transacciones multi-statement; no está pensado para OLTP |

Leer de una **read replica** (RDS, Aurora, Cloud SQL) es otro tema aparte del isolation: la réplica puede ir atrasada respecto del primario, así que una lectura ahí puede devolver datos más viejos que los de una transacción que ya hizo commit en el primario.

Ver también [Locks (shared/exclusive/MVCC)](locks.es.md) — el mecanismo interno que hace cumplir estos niveles — y [Rollback / savepoints](rollback-savepoints.es.md).
