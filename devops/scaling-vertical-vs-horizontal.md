# Vertical vs Horizontal Scalability

The two ways to give a system more capacity.

**Vertical**: add more CPU/RAM to the same machine. Simple — it doesn't change anything about the architecture — but it has a physical ceiling (there's no infinitely large instance) and almost always involves downtime when resizing (a reboot is needed).

**Horizontal**: add more instances running in parallel. No theoretical ceiling, but it requires the app to be *stateless* (session in a shared store, not in the process's memory — see the example in [Scalability](../system-design/quality-attributes.md#scalability)) and needs a load balancer distributing traffic across instances.

| | Vertical | Horizontal |
|---|---|---|
| How it's done | More CPU/RAM on the same machine | More instances in parallel |
| Ceiling | Physical (the largest instance size that exists) | Theoretically none |
| Downtime when scaling | Yes, almost always (reboot) | No, another instance is added without touching the ones already running |
| Prerequisite | None | Stateless app + load balancer |

In practice they're combined: each instance in a horizontal cluster also has its own (vertical) size chosen for it.

---
Related: [Scalability](../system-design/quality-attributes.md#scalability), [DevOps Diagnostics](../diagnostics/devops.md).
