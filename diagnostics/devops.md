# DevOps Diagnostics

Most common causes of infrastructure-level slowness, from most to least frequent.

## Lack of vertical scalability

The server has gotten too small for the current load (CPU/RAM maxed out) and nobody bumped up the instance size. Before adding complexity with more nodes, confirm whether scaling vertically solves the current bottleneck. See [Vertical vs horizontal scalability](../devops/scaling-vertical-vs-horizontal.md).

## Poor load balancing in K8s

Pods without properly configured **readiness probes** receive traffic before they're ready; missing **HPA** (Horizontal Pod Autoscaler) means load doesn't get redistributed when replicas are added; badly set `resource limits` cause silent CPU throttling. See [Kubernetes](../devops/kubernetes.md).

```yaml
# ✅ readinessProbe: doesn't receive traffic until it responds 200 on /health
readinessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 10
resources:
  requests: { cpu: "250m", memory: "256Mi" }
  limits: { cpu: "500m", memory: "512Mi" }
```

## Slow network / missing CDN

See [CDN](../devops/cdn.md). Static assets (images, JS, CSS) served straight from the origin force every user to travel all the way to wherever that server is — if the origin is in the US and the user is in Argentina, every request drags that geographic latency along even though the file never changes. A CDN caches copies on servers (*edge nodes*) close to each user, so the file is served from the nearest point instead of crossing the world on every request.

```
# ✅ tells the CDN it can cache the asset for 1 year (files with a hash in the name
# never change, so a long cache is safe)
Cache-Control: public, max-age=31536000, immutable
```

## Cold start

Serverless functions (Lambda, Cloud Functions) or containers that scale to zero when there's no traffic take time to start up on the first request after being idle: they have to initialize the runtime, load dependencies, and sometimes establish connections (DB, etc.) before responding. Shows up as high, intermittent latency on sporadic requests. Typical mitigations: *provisioned concurrency*, keeping a minimum instance always warm, or shrinking the package/dependency size to start up faster. See [Cold starts](../devops/cold-starts.md).

```js
// ✅ Node Lambda: DB connection outside the handler, reused across "warm" invocations
const db = connectToDb(); // runs only once per cold start, not on every request

export const handler = async (event) => {
  const result = await db.query('SELECT ...');
  return { statusCode: 200, body: JSON.stringify(result) };
};
```
