# Webhooks

The inverse of a normal API: instead of you asking it ("did anything happen?"), the other system **notifies** you as soon as it does, sending a `POST` to a URL you expose.

## Push vs polling

Without webhooks, to find out about an event in an external system (a payment that got confirmed, a file that finished processing) you have to **poll**: repeatedly asking "is it done yet?" until the answer is yes — wastes requests most of the time, and there's latency between when the event happened and when you detected it (depends on how often you ask).

```python
# ❌ polling: most of these calls return "not yet"
while not payment_confirmed(payment_id):
    time.sleep(5)

# ✅ webhook: your endpoint receives the POST as soon as the event happens, without asking anything
@app.post("/webhooks/stripe")
async def stripe_webhook(request: Request):
    event = await request.json()
    if event["type"] == "payment_intent.succeeded":
        mark_payment_confirmed(event["data"]["id"])
```

## Verify the signature — don't trust the body blindly

Anyone who knows your webhook URL could send it a fake `POST` simulating a real event (e.g. "the payment was confirmed" when it wasn't). Serious providers sign every request with **HMAC** using a shared secret — your endpoint recomputes the signature over the received body and compares it to the header the provider sent, before trusting the content.

```python
import hmac, hashlib

def verify_webhook_signature(payload: bytes, signature_header: str, secret: str) -> bool:
    expected = hmac.new(secret.encode(), payload, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, signature_header)  # safe comparison against timing attacks
```

## Idempotency — the same webhook can arrive duplicated

If your endpoint doesn't respond in time (or responds with an error), the sender usually **retries** the same webhook later — your handler has to be able to process it twice without duplicating the effect (see [Idempotency](../system-design/quality-attributes.md#idempotency)). Most providers include a unique event ID in the payload — save it and check whether it was already processed before applying the effect again.

```python
def handle_webhook(event):
    if already_processed(event["id"]):  # ✅ avoids duplicating the effect on a retry
        return
    apply_effect(event)
    mark_as_processed(event["id"])
```

## Typical cases

Payments (Stripe, MercadoPago notify when a charge is confirmed), CI/CD (GitHub notifies an external service on a push or a PR), integrations (Slack, Discord notifying a bot of events).

---
Related: [Idempotency](../system-design/quality-attributes.md#idempotency), [WebSocket / SSE / Streaming](../frontend-react/websocket-sse-streaming.md) (another way to receive data without polling, but with a persistent connection instead of one request per event).
