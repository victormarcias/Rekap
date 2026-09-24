# Message Queues: SQS vs RabbitMQ vs AMQP

[Kafka Architecture](kafka.md) already covers the **distributed log** model. This is the same problem (asynchronous/decoupled communication between services) solved with **queues** — a different model, with its own tools.

## AMQP isn't a product — it's a protocol

**AMQP** (Advanced Message Queuing Protocol) is an open standard that defines how producers, queues, and consumers communicate — it's not a tool you install, it's the specification that **RabbitMQ implements**. Confusing "AMQP" with "RabbitMQ" is common: RabbitMQ is *one* broker that speaks AMQP, not the only one possible.

## RabbitMQ

Traditional message broker, with **flexible routing** via *exchanges*: a message doesn't go straight to a queue, it goes to an exchange that decides (based on rules) which queue(s) to forward it to.

- **Direct exchange**: routes by an exact key (like a switch).
- **Topic exchange**: routes by patterns (`orders.created.*`).
- **Fanout exchange**: sends a copy of the message to **all** connected queues — the equivalent of a broadcast.

It's *push-based*: the broker pushes the message to the consumer as soon as it's available.

## SQS (Amazon Simple Queue Service)

A queue fully managed by AWS — no server of your own to run or maintain. A much simpler model than RabbitMQ: basically point to point, without the exchange/routing system.

It's *pull-based*: the consumer *polls* the queue asking "is there anything?" instead of the queue pushing the message to it — with **long polling** (waiting a few seconds before responding empty) so it doesn't waste requests on constant polling with nothing new.

```
# Conceptual — the consumer actively asks, it doesn't wait to be notified
while True:
    messages = sqs.receive_message(QueueUrl=queue_url, WaitTimeSeconds=20)  # long polling
    for msg in messages:
        process(msg)
        sqs.delete_message(msg)  # only now is it confirmed as processed
```

## When to use a queue (SQS/RabbitMQ) vs a log (Kafka)

| | Queue (SQS/RabbitMQ) | Log (Kafka) |
|---|---|---|
| Message after being consumed | Deleted | Kept (configurable retention) |
| Multiple consumers of the same message | Requires explicit fan-out (fanout exchange, or one queue per consumer) | Each consumer group reads everything, independently |
| Replay | ❌ | ✅ |
| Typical use case | Work tasks (processing an order, sending an email) — each task is handled by **a single** worker | Event streaming, when several systems need to see the same event independently |

See the full queue vs log comparison in [Kafka Architecture](kafka.md#kafka-vs-traditional-queue).

---
Related: [Kafka Architecture](kafka.md), [Idempotency](../system-design/atributos-de-calidad.md#idempotencia) (a message can be reprocessed after a network failure — the consumer needs to be idempotent).
