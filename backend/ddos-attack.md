# DDoS Attack

**Distributed Denial of Service**: multiple sources (often a botnet — thousands of compromised machines) flood a service with traffic, trying to exhaust its resources (CPU, network, connections) until it becomes unreachable for legitimate users. "Distributed" is the key word — unlike a single-source DoS, blocking one IP isn't enough.

## Types, from easiest to hardest to mitigate

- **Volumetric**: directly saturates the available bandwidth with a massive volume of traffic — it doesn't matter if the requests are valid, the network link fills up before they get processed.
- **Protocol**: exploits a protocol's behavior instead of bandwidth. The classic is the **SYN flood**: sending thousands of `SYN`s (the first step of the [three-way handshake](../system-design/what-happens-when-you-type-a-url.md#3-tcp-connection--three-way-handshake)) without ever completing the final `ACK` — the server reserves resources for each "half-open" connection until it exhausts them.
- **Application**: completely legitimate-looking HTTP requests, but at a volume that saturates the backend's processing capacity — the hardest to distinguish from a real traffic spike, because each individual request looks normal.

## Mitigations by layer

- **CDN/edge**: absorbs volumetric traffic before it reaches the origin — see [CDN](../devops/cdn.md). Most of the traffic from a large attack doesn't even reach your real infrastructure.
- **Rate limiting**: caps how many requests it accepts per IP/client in a time window, returning [429 Too Many Requests](../system-design/http-status-codes.md#4xx--client-error) to the rest — usually centralized at the [API Gateway](api-gateway.md), not in each service.
- **WAF (Web Application Firewall)**: filters known malicious traffic patterns before they reach the application.
- **Autoscaling — with care**: scaling automatically in response to a traffic spike helps absorb the attack, but without a ceiling it can turn into an **"economic denial of service attack"**: the attacker doesn't take the service down, but racks up a huge cloud bill by forcing infrastructure to scale that never served real traffic. See [Elasticity](../system-design/quality-attributes.md#elasticity) — always with a `maxReplicas` (or equivalent) as a hard ceiling.

## The hard part: telling an attack from legitimate traffic

Content that goes viral generates a traffic pattern similar to an application attack — lots of volume, from many different sources, all requesting the same thing. The difference is usually in the details: an attack tends to have more uniform/artificial patterns (same user-agent, same intervals, geographies unusual for the business) than a real organic spike — but the line isn't always sharp, which is why rate limiting and WAFs work with heuristics, not certainties.

---
Related: [CDN](../devops/cdn.md), [API Gateway](api-gateway.md), [What happens when you type a URL](../system-design/what-happens-when-you-type-a-url.md) (TCP handshake), [Elasticity](../system-design/quality-attributes.md#elasticity), [Security](../security/) (application vulnerabilities, different from a traffic attack).
