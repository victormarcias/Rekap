# Kafka Architecture

Apache Kafka is a distributed streaming platform — the central piece for decoupling services that need to communicate through events instead of direct calls.

## What Kafka is

Unlike a traditional message queue (RabbitMQ, SQS), Kafka is an **append-only distributed log**: messages are written sequentially and kept for a configurable time/size — **they aren't deleted when consumed**. Any consumer can read from any point in the log, even messages another consumer already read a while ago.

## Topics and partitions

A **topic** is a named channel that messages get published to (e.g. `order_events`). Internally, each topic is split into **partitions** — the actual log is per partition, not per topic. Kafka guarantees order **within** a partition, but **not across different partitions** of the same topic: if the order between two messages matters, they have to go to the same partition (typically by choosing the partition based on a key, so all related events stay together and ordered).

```python
# the same order_id always goes to the same partition → its events stay ordered relative to each other
producer.send('order_events', key=str(order_id), value=event_data)
```

## Producers / Consumers and Consumer Groups

**Producers** publish messages to a topic; **consumers** read them. Several consumers can be grouped into a **consumer group**: Kafka splits the topic's partitions among the consumers in that group, so each message is processed by **only one consumer in the group** — this is how consumption scales horizontally, adding consumers to the group up to a maximum of one consumer per partition.

```python
consumer = KafkaConsumer('order_events', group_id='inventory-service')
# with 4 partitions and 2 consumers in the group, each consumer handles 2 partitions
```

If two different consumer groups read the same topic (e.g. `inventory-service` and `analytics-service`), **each group** gets a full copy of every message — they're independent of each other.

## Offset

Each consumer tracks its position in each partition with an **offset** (a number that advances sequentially). Since messages aren't deleted when read, a consumer can resume from the last saved offset if it goes down (no messages lost), or rewind the offset by hand to reprocess old messages (e.g. rebuild state after a bug). This is what enables **replay** — something a traditional queue doesn't offer, because there a message disappears as soon as it's `ack`ed.

## Kafka vs traditional queue

| | Traditional queue (RabbitMQ, SQS) | Kafka |
|---|---|---|
| Message after being consumed | Deleted | Kept (per configured retention) |
| Multiple consumers of the same message | Requires explicit fan-out | Each consumer group reads everything, independently |
| Replay | ❌ | Yes (rewind the offset) |
| Guaranteed order | Per queue | Per partition |

---
Related: [Idempotency](../system-design/atributos-de-calidad.md#idempotencia) (a consumer can reprocess the same message more than once after a crash — the handler needs to be idempotent), [Observability](../system-design/atributos-de-calidad.md#observabilidad).
