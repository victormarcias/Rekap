# Cold starts

Extra latency paid by the first request when a serverless function or a container that [scales to zero](vps-vs-cloud-run.md) has to start from scratch: initialize the runtime, load dependencies, and sometimes open connections (DB, etc.) before it can respond. Shows up as intermittent latency spikes on sporadic traffic — subsequent requests, while the instance stays "warm," don't pay that cost.

**Typical mitigations**:
- *Provisioned concurrency* — pay to keep N instances always warm, sacrificing part of the savings from scaling to zero.
- Shrink the package/image size and the dependencies loaded at startup.
- Open connections (DB, HTTP clients) outside the handler, so they're reused across warm invocations instead of being recreated on every cold start — see the example in [DevOps Diagnostics](../diagnostics/devops.md).

---
Related: [DevOps Diagnostics](../diagnostics/devops.md), [Deploy to Cloud Run](deploy-cloud-run.md) (scale to zero), [Elasticity](../system-design/atributos-de-calidad.md#elasticidad).
