# Webhooks

El inverso de una API normal: en vez de que vos le preguntes ("¿pasó algo nuevo?"), el otro sistema te **avisa** apenas pasa, mandando un `POST` a una URL que vos exponés.

## Push vs polling

Sin webhooks, para enterarte de un evento en un sistema externo (un pago que se acreditó, un archivo que terminó de procesarse) tenés que hacer **polling**: preguntar repetidamente "¿ya está?" hasta que la respuesta sea sí — desperdicia requests la mayor parte del tiempo, y hay latencia entre que el evento ocurrió y que lo detectaste (depende de cada cuánto preguntás).

```python
# ❌ polling: la mayoría de estas llamadas devuelven "todavía no"
while not payment_confirmed(payment_id):
    time.sleep(5)

# ✅ webhook: tu endpoint recibe el POST apenas el evento ocurre, sin preguntar nada
@app.post("/webhooks/stripe")
async def stripe_webhook(request: Request):
    event = await request.json()
    if event["type"] == "payment_intent.succeeded":
        mark_payment_confirmed(event["data"]["id"])
```

## Verificar la firma — no confiar en el body a ciegas

Cualquiera que conozca tu URL de webhook podría mandarle un `POST` falso simulando un evento real (ej. "el pago se confirmó" cuando no fue así). Los proveedores serios firman cada request con **HMAC** usando un secreto compartido — tu endpoint recalcula la firma sobre el body recibido y la compara con el header que mandó el proveedor, antes de confiar en el contenido.

```python
import hmac, hashlib

def verify_webhook_signature(payload: bytes, signature_header: str, secret: str) -> bool:
    expected = hmac.new(secret.encode(), payload, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, signature_header)  # comparación segura contra timing attacks
```

## Idempotencia — el mismo webhook puede llegar duplicado

Si tu endpoint no responde a tiempo (o responde con un error), el emisor suele **reintentar** el mismo webhook más tarde — tu handler tiene que poder procesarlo dos veces sin duplicar el efecto (ver [Idempotencia](../system-design/quality-attributes.es.md#idempotencia)). La mayoría de los proveedores incluyen un ID de evento único en el payload — guardarlo y chequear si ya se procesó antes de aplicar el efecto de nuevo.

```python
def handle_webhook(event):
    if already_processed(event["id"]):  # ✅ evita duplicar el efecto en un reintento
        return
    apply_effect(event)
    mark_as_processed(event["id"])
```

## Casos típicos

Pagos (Stripe, MercadoPago avisan cuando se confirma un cobro), CI/CD (GitHub avisa a un servicio externo cuando hay un push o un PR), integraciones (Slack, Discord notificando eventos a un bot).

---
Relacionado: [Idempotencia](../system-design/quality-attributes.es.md#idempotencia), [WebSocket / SSE / Streaming](../frontend-react/websocket-sse-streaming.es.md) (otra forma de recibir datos sin polling, pero con conexión persistente en vez de un request puntual por evento).
