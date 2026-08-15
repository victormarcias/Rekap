# Key-Value Store / DynamoDB

[NoSQL](../database/nosql.md) ya cubre Key-Value store como categoría general. Esto es DynamoDB en concreto — el ejemplo más usado del modelo, y sus particularidades.

## Partition key y sort key

Cada tabla tiene una **partition key** (obligatoria) que determina en qué partición física vive el ítem — DynamoDB la hashea para distribuir los datos entre nodos, similar en espíritu a [Sharding](../database/sharding-vs-partitioning.md#sharding). Opcionalmente, una **sort key** permite ordenar/filtrar múltiples ítems dentro de la misma partition key.

```
# partition key sola: cada usuario es una partición
users: { partition_key: user_id }

# partition key + sort key: todas las órdenes de un usuario viven en la misma partición,
# ordenadas por fecha — "traer las últimas 10 órdenes de este usuario" es barato
orders: { partition_key: user_id, sort_key: created_at }
```

Elegir mal la partition key es el error más común: si un valor concentra demasiado tráfico (ej. una partition key con pocos valores distintos, o uno "caliente" que todo el mundo pide), esa partición se convierte en cuello de botella — DynamoDB no puede paralelizar dentro de una sola partición.

## Sin joins — la denormalización es obligatoria

A diferencia de SQL (ver [Normalización](../database/normalizacion.md)), DynamoDB no soporta joins entre tablas. El modelado se hace al revés que en relacional: en vez de normalizar y hacer join al leer, se **denormaliza a propósito**, duplicando datos para que cada patrón de acceso frecuente se resuelva con una sola query a una sola tabla. El diseño del schema arranca por "¿qué preguntas le voy a hacer a esta tabla?", no por "¿cómo represento la entidad de forma pura?".

## Capacidad: provisioned vs on-demand

- **Provisioned**: se reserva de antemano cuánta lectura/escritura por segundo va a soportar la tabla — más barato si el tráfico es predecible, pero un pico inesperado puede ser *throttled* (rechazado) si supera lo reservado.
- **On-demand**: escala automáticamente con el tráfico real, sin reservar nada — más caro por unidad, pero sin necesidad de estimar capacidad de antemano ni riesgo de throttling por subestimarla.

## Cuándo tiene sentido

Patrones de acceso simples y conocidos de antemano (buscar por id, o por id + rango), volumen alto, necesidad de escalar horizontalmente sin administrar servidores. No es la herramienta si las queries van a necesitar filtros/joins ad-hoc que no se pueden prever al diseñar el schema — ahí [SQL](../database/README.md) sigue siendo mejor opción.

---
Relacionado: [NoSQL](../database/nosql.md), [Sharding vs partitioning](../database/sharding-vs-partitioning.md), [Normalización](../database/normalizacion.md) (denormalización).
