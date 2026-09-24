# Escalabilidad de Base de Datos

[Sharding vs partitioning](sharding-vs-partitioning.es.md) ya cubre a fondo cómo escalar distribuyendo los **datos**. Esto completa el panorama con la otra estrategia: escalar distribuyendo las **lecturas**.

## Read replicas

Copias de solo lectura de la base principal, sincronizadas por replicación (normalmente asíncrona). Las escrituras siguen yendo todas al nodo principal (*primary*/*master*); las lecturas se reparten entre las réplicas — en la mayoría de las apps reales, las lecturas superan ampliamente a las escrituras, así que escalar lecturas de esta forma resuelve gran parte de la carga sin tocar cómo se particionan los datos.

```
                    ┌──────────┐
   escrituras  ───→ │ Primary  │
                    └────┬─────┘
                         │ replicación asíncrona
              ┌──────────┼──────────┐
              ▼          ▼          ▼
         ┌─────────┐┌─────────┐┌─────────┐
lecturas → │Replica 1││Replica 2││Replica 3│
         └─────────┘└─────────┘└─────────┘
```

**El costo**: replicación asíncrona implica *replication lag* — una réplica puede estar unos milisegundos (o más, bajo carga) atrás del primary. Leer de una réplica justo después de escribir en el primary puede devolver el dato viejo todavía no replicado — el mismo tipo de trade-off consistencia-vs-disponibilidad de [CAP Theorem](nosql.es.md#cap-theorem), aplicado dentro de una base relacional. Para el caso "necesito leer exactamente lo que acabo de escribir", hay que leer del primary a propósito, no de una réplica.

## Procesar resultados grandes en chunks

Traer una tabla de millones de filas con un solo `fetchall()` carga **todo** el resultado en memoria de una — el límite ya no es la base de datos, es la RAM del proceso que está leyendo (ver [Escalabilidad de Memoria](../devops/scaling-memory.es.md)). La solución no es traer menos datos, es traer los mismos datos **de a partes**: pedir un chunk, procesarlo, descartarlo, pedir el siguiente — la memoria usada queda acotada al tamaño del chunk, sin importar cuán grande sea la tabla completa.

```python
# ❌ carga el resultado completo en memoria — con una tabla grande, esto puede OOMear el proceso
rows = cursor.execute("SELECT * FROM huge_table").fetchall()
for row in rows:
    process(row)

# ✅ trae de a chunks: cada lote se procesa y se libera antes de pedir el siguiente
cursor.execute("SELECT * FROM huge_table")
while True:
    chunk = cursor.fetchmany(1000)
    if not chunk:
        break
    for row in chunk:
        process(row)
    # el chunk anterior queda libre para el garbage collector acá, antes del próximo fetchmany
```

Muchos drivers también soportan un **cursor del lado del servidor** (*server-side cursor* — en Postgres, un named cursor): en vez de que el driver traiga toda la respuesta y la vaya entregando de a partes desde el cliente, es la propia base la que retiene el resultado y lo va enviando en streaming — ahorra memoria también del lado de la DB, no solo del lado de la app. Caso de uso típico: procesos de ETL/exportación que recorren una tabla entera sin necesitar tenerla completa en memoria en ningún momento.

## Vertical Partitioning

El [partitioning](sharding-vs-partitioning.es.md#partitioning) horizontal divide **filas** (misma tabla, mismas columnas, distintas particiones); el vertical divide **columnas**: la tabla se separa en dos o más tablas más angostas, cada una con un subconjunto de columnas, unidas por la misma clave primaria.

La señal típica de que hace falta: una tabla ancha, con columnas que casi nunca se usan juntas, muchas de ellas `NULL` la mayor parte del tiempo porque en realidad guardan conceptos distintos (datos de contacto, demográficos, info de facturación) pegoteados en una sola fila.

```sql
-- ❌ una tabla mezclando datos de uso frecuente con datos poco usados/opcionales
CREATE TABLE person (
  id bigint PRIMARY KEY,
  first_name text, last_name text,        -- se leen en casi cada query
  additional_contact_info xml,             -- casi siempre NULL, pesado, poco usado
  demographics xml,                        -- casi siempre NULL, pesado, poco usado
  credit_card_id bigint                     -- solo aplica a algunos registros
);

-- ✅ separado por columnas: lo frecuente queda liviano, lo pesado/opcional se consulta aparte
CREATE TABLE person (
  id bigint PRIMARY KEY,
  first_name text, last_name text
);
CREATE TABLE person_details (
  person_id bigint PRIMARY KEY REFERENCES person(id),
  additional_contact_info xml,
  demographics xml
);
```

**Por qué ayuda**: una query que solo necesita `first_name`/`last_name` (la mayoría) ahora lee filas más chicas — más filas entran en cada página de disco, más filas caben en el buffer pool en memoria (ver [Escalabilidad de Memoria](../devops/scaling-memory.es.md)) — sin tener que arrastrar columnas pesadas u opcionales que ni siquiera pidió.

**El límite**: si la dispersión no es "un puñado de grupos de columnas relacionadas" sino que es genuinamente **variable por fila** (cada registro necesita un set de campos distinto e impredecible de antemano), seguir partiendo verticalmente no alcanza — ahí es donde conviene evaluar [NoSQL](nosql.es.md) (un document store) en vez de forzar más el modelo relacional.

## Tabla activa vs histórica

Cuando la mayoría de las queries solo le importan los registros "recientes" (facturas abiertas, pedidos del último mes) pero la tabla acumula años de datos cerrados, mover lo viejo a una tabla separada (`facturas_history`) mantiene la tabla activa chica — índices más chicos y rápidos, backups/mantenimiento más baratos, y las queries del día a día no tienen que atravesar años de registros que nunca van a necesitar.

```sql
-- mover facturas cerradas hace más de 2 años a una tabla histórica separada
BEGIN;
INSERT INTO facturas_history SELECT * FROM facturas WHERE created_at < now() - interval '2 years';
DELETE FROM facturas WHERE created_at < now() - interval '2 years';
COMMIT;
```

**Diferencia con [range partitioning](sharding-vs-partitioning.es.md#partitioning)**: el partitioning es transparente para las queries — un `SELECT` sobre `facturas` sigue funcionando igual sin importar cuántas particiones haya por detrás. Separar en `facturas`/`facturas_history` es una separación explícita a nivel de aplicación: el código que necesita datos viejos tiene que *saber* que existe la segunda tabla y consultarla a propósito. Es más simple de implementar (no requiere configurar partitioning en el motor), pero menos transparente.

## Snapshot tables vs vistas materializadas

Las dos precalculan y persisten el resultado de una agregación costosa, pero difieren en quién controla el refresh:

- **Vista materializada**: el motor de la DB la refresca (`REFRESH MATERIALIZED VIEW`) — sigue siendo un objeto nativo de la base, con su propio manejo interno.
- **Snapshot table**: una tabla común, poblada por un job externo (cron, worker) que corre el cálculo y hace `INSERT`/`UPDATE` — control total sobre cuándo y cómo se refresca (ej. incremental en vez de recalcular todo desde cero), a costa de mantener ese job vos mismo.

```sql
-- ✅ snapshot table: un job corre esto todas las noches, no la propia DB
CREATE TABLE ventas_diarias_snapshot (
  fecha date PRIMARY KEY,
  total numeric
);

-- el job (cron/worker) hace este upsert cada noche — refresh incremental, no recalcula todo
INSERT INTO ventas_diarias_snapshot (fecha, total)
SELECT current_date, SUM(total) FROM ventas WHERE created_at::date = current_date
ON CONFLICT (fecha) DO UPDATE SET total = EXCLUDED.total;
```

**Cuándo usar cada uno**: vista materializada cuando el motor ya soporta el tipo de refresh que necesitás, sin código de aplicación extra. Snapshot table cuando hace falta un refresh incremental, coordinado con otros pasos de un pipeline, o cuando el motor ni siquiera soporta vistas materializadas nativas (MySQL, por ejemplo, no las tiene — se resuelve con snapshot tables).

## Vertical vs horizontal, aplicado a DB

- **Vertical**: más CPU/RAM/IOPS a la misma instancia de DB — el default más simple, con techo físico.
- **Horizontal (lecturas)**: read replicas, como arriba.
- **Horizontal (datos)**: [sharding](sharding-vs-partitioning.es.md#sharding) — cuando ni escalar vertical ni sumar réplicas de lectura alcanza, porque el problema es volumen de datos/escritura, no solo lecturas.

Read replicas y sharding no son excluyentes — un sistema grande típicamente combina los dos: varios shards, cada uno con sus propias réplicas de lectura.

---
Relacionado: [Sharding vs partitioning](sharding-vs-partitioning.es.md), [Consistencia](../system-design/quality-attributes.es.md#consistencia), [Connection pooling](../diagnostics/backend.es.md#conexiones-mal-gestionadas), [Escalabilidad de Memoria](../devops/scaling-memory.es.md), [NoSQL](nosql.es.md).
