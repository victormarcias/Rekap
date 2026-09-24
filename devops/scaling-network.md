# Network Scalability

Network capacity at the infrastructure level — different from *chattiness* at the code level (many small calls instead of a few big ones), already covered in [HTTP chaining](../diagnostics/backend.md#http-chaining). Here the problem isn't how your code calls out, it's that the **available bandwidth** is already saturated, no matter how efficient the call is.

## When the bottleneck is the network, not the code

Shows up as relaxed CPU and memory, but low throughput and high latency — the process isn't busy computing, it's waiting for data to finish traveling. Typical in large transfers (backups, DB replicas, file streaming) or in high east-west traffic between many microservices in the same region.

## Scaling network capacity

- **Instances with more dedicated bandwidth**: in the cloud, network bandwidth usually scales along with instance size (a bigger instance doesn't just have more CPU/RAM, it also has more Gbps available) — sometimes the real "CPU bottleneck" is actually a small instance's network limit.
- **CDN for outbound traffic to users**: moving static asset traffic off the origin server to the edge directly reduces the network load your infrastructure has to handle — see [CDN](cdn.md).
- **Compression**: fewer bytes traveling over the same bandwidth — `gzip`/`br` on responses reduces network pressure without changing infrastructure.
- **Bringing services that talk to each other close together**: two services in different regions that call each other often pay high network latency on every call — placing them in the same region/availability zone reduces that latency and the inter-region traffic (which is also usually billed separately).

---
Related: [CDN](cdn.md), [HTTP chaining](../diagnostics/backend.md#http-chaining), [Vertical vs Horizontal Scalability](scaling-vertical-vs-horizontal.md).
