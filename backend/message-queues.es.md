# Colas de mensajes: SQS vs RabbitMQ vs AMQP

[Arquitectura Kafka](kafka.es.md) ya cubre el modelo de **log distribuido**. Esto es lo mismo problema (comunicación asíncrona/desacoplada entre servicios) resuelto con **colas** — un modelo distinto, con sus propias herramientas.

## AMQP no es un producto — es un protocolo

**AMQP** (Advanced Message Queuing Protocol) es un estándar abierto que define cómo se comunican productores, colas y consumers — no es una herramienta que se instala, es la especificación que **RabbitMQ implementa**. Confundir "AMQP" con "RabbitMQ" es común: RabbitMQ es *un* broker que habla AMQP, no el único posible.

## RabbitMQ

Message broker tradicional, con **routing flexible** vía *exchanges*: un mensaje no va directo a una cola, va a un exchange que decide (según reglas) a qué cola(s) reenviarlo.

- **Direct exchange**: rutea por una key exacta (como un switch).
- **Topic exchange**: rutea por patrones (`orders.created.*`).
- **Fanout exchange**: le pega una copia del mensaje a **todas** las colas conectadas — el equivalente a un broadcast.

Es *push-based*: el broker empuja el mensaje al consumer apenas está disponible.

## SQS (Amazon Simple Queue Service)

Cola totalmente administrada por AWS — no hay servidor propio que correr ni mantener. Modelo mucho más simple que RabbitMQ: básicamente punto a punto, sin el sistema de exchanges/routing.

Es *pull-based*: el consumer hace *polling* a la cola preguntando "¿hay algo?" en vez de que la cola le empuje el mensaje — con **long polling** (esperar unos segundos antes de responder vacío) para no gastar requests en polling constante sin nada nuevo.

```
# Conceptual — el consumer pregunta activamente, no espera a que le avisen
while True:
    messages = sqs.receive_message(QueueUrl=queue_url, WaitTimeSeconds=20)  # long polling
    for msg in messages:
        process(msg)
        sqs.delete_message(msg)  # recién acá se confirma que se procesó
```

## Cuándo cola (SQS/RabbitMQ) vs cuándo log (Kafka)

| | Cola (SQS/RabbitMQ) | Log (Kafka) |
|---|---|---|
| Mensaje tras consumirse | Se borra | Se conserva (retención configurable) |
| Múltiples consumers del mismo mensaje | Requiere fan-out explícito (fanout exchange, o una cola por consumer) | Cada consumer group lee todo, independiente |
| Replay | ❌ | ✅ |
| Caso de uso típico | Tareas de trabajo (procesar una orden, mandar un email) — cada tarea la resuelve **un solo** worker | Streaming de eventos, cuando varios sistemas necesitan ver el mismo evento de forma independiente |

Ver la comparación completa cola vs log en [Arquitectura Kafka](kafka.es.md#kafka-vs-cola-tradicional).

---
Relacionado: [Arquitectura Kafka](kafka.es.md), [Idempotencia](../system-design/quality-attributes.es.md#idempotencia) (un mensaje puede reprocesarse tras un fallo de red — el consumer necesita ser idempotente).
