# CDN (Content Delivery Network)

Geographically distributed network of servers (*edge nodes*) that cache copies of static assets close to each user, so requests don't have to travel all the way to the origin server every time.

Without a CDN, a user in Argentina requesting an image from a server in the US pays that geographic latency on **every** request, even though the file never changes. With a CDN, the first visit from a region caches the file at the nearest edge node; every other user in that region gets it from there.

```
Cache-Control: public, max-age=31536000, immutable
```

That header tells the CDN (and the browser) it can cache the asset for a year — safe when the file has a hash in its name (`app.a1b2c3.js`), so a content change always produces a new URL instead of silently updating the old one.

**Invalidation**: if you need to force the CDN to drop a stale copy before its TTL expires (e.g. an urgent deploy), that's done with an explicit *purge*/*invalidation* against the provider's API — it's the exception, not the normal flow.

---
Related: [DevOps Diagnostics](../diagnostics/devops.md) (symptom: slow network due to missing CDN).
