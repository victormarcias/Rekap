# Escalabilidad de Base de Datos

[Sharding vs partitioning](sharding-vs-partitioning.md) ya cubre a fondo cómo escalar distribuyendo los **datos**. Esto completa el panorama con la otra estrategia: escalar distribuyendo las **lecturas**.

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

**El costo**: replicación asíncrona implica *replication lag* — una réplica puede estar unos milisegundos (o más, bajo carga) atrás del primary. Leer de una réplica justo después de escribir en el primary puede devolver el dato viejo todavía no replicado — el mismo tipo de trade-off consistencia-vs-disponibilidad de [CAP Theorem](nosql.md#cap-theorem), aplicado dentro de una base relacional. Para el caso "necesito leer exactamente lo que acabo de escribir", hay que leer del primary a propósito, no de una réplica.

## Procesar resultados grandes en chunks

Traer una tabla de millones de filas con un solo `fetchall()` carga **todo** el resultado en memoria de una — el límite ya no es la base de datos, es la RAM del proceso que está leyendo (ver [Escalabilidad de Memoria](../devops/escalabilidad-memoria.md)). La solución no es traer menos datos, es traer los mismos datos **de a partes**: pedir un chunk, procesarlo, descartarlo, pedir el siguiente — la memoria usada queda acotada al tamaño del chunk, sin importar cuán grande sea la tabla completa.

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

## Vertical vs horizontal, aplicado a DB

- **Vertical**: más CPU/RAM/IOPS a la misma instancia de DB — el default más simple, con techo físico.
- **Horizontal (lecturas)**: read replicas, como arriba.
- **Horizontal (datos)**: [sharding](sharding-vs-partitioning.md#sharding) — cuando ni escalar vertical ni sumar réplicas de lectura alcanza, porque el problema es volumen de datos/escritura, no solo lecturas.

Read replicas y sharding no son excluyentes — un sistema grande típicamente combina los dos: varios shards, cada uno con sus propias réplicas de lectura.

---
Relacionado: [Sharding vs partitioning](sharding-vs-partitioning.md), [Consistencia](../system-design/atributos-de-calidad.md#consistencia), [Connection pooling](../diagnostico/backend.md#conexiones-mal-gestionadas), [Escalabilidad de Memoria](../devops/escalabilidad-memoria.md).
