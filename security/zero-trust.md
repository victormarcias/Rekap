# Zero Trust

Modelo de seguridad: **nunca confiar, siempre verificar**. Cada request se autentica y autoriza, sin importar de dónde venga — no existe una zona "de adentro" automáticamente confiable.

## El modelo que reemplaza: perímetro (castillo y foso)

El modelo clásico concentra la seguridad en el borde de la red — firewall, VPN — y una vez que algo pasó ese borde, se mueve con confianza amplia y poca re-verificación. Funciona mientras el borde no se rompa, pero falla exactamente cuando más importa: una credencial de VPN robada, un empleado comprometido, o un servicio interno vulnerado le dan a un atacante movimiento libre puertas adentro, porque el diseño nunca esperó tener que desconfiar de "adentro".

## Principios centrales

- **Verificar explícitamente**: cada request se autentica y autoriza — sin importar si viene de internet o de la red interna. "Interno" deja de ser sinónimo de "confiable".
- **Mínimo privilegio**: acceso acotado a lo estrictamente necesario para esa identidad puntual, nunca de más "por las dudas" — mismo criterio que ya vimos en [SQL Injection](sql-injection.md#cómo-se-previene) a nivel de DB, o en los permisos de un [agente de IA](../agentic-ai/riesgos-y-mitigaciones.md#mitigaciones-técnicas).
- **Asumir la brecha**: diseñar como si el atacante ya estuviera adentro — segmentar la red, cifrar tráfico interno además del externo, monitorear activamente en vez de confiar en que el perímetro aguantó.

## Ejemplo: autenticar también el tráfico "interno"

```python
# ❌ modelo de perímetro: confía en cualquier request que venga de la red interna
@app.get("/orders/{order_id}")
def get_order(order_id: str, request: Request):
    if es_ip_interna(request.client.host):   # "ya pasó el firewall, no hace falta más"
        return buscar_orden(order_id)
    raise HTTPException(403)

# ✅ Zero Trust: valida identidad en cada request, sin importar el origen
@app.get("/orders/{order_id}")
def get_order(order_id: str, token: str = Depends(verificar_token)):
    if not tiene_permiso(token, "orders:read"):
        raise HTTPException(403)
    return buscar_orden(order_id)
```

La diferencia no es cosmética: en el primer caso, cualquiera que logre pararse dentro de la red (VPN robada, un contenedor comprometido en la misma VPC) accede sin más preguntas. En el segundo, necesita además un token válido con el permiso específico — la red por sí sola no alcanza.

## mTLS entre microservicios

La misma idea aplicada a comunicación servicio-a-servicio: con **mTLS** (mutual TLS) cada servicio presenta su propio certificado, y el que recibe la conexión verifica la identidad del que llama — no solo al revés (cliente verificando servidor, como en HTTPS normal). Dos servicios en la misma red privada igual se autentican entre sí en cada llamado, en vez de asumir que "están en la misma VPC" ya es suficiente garantía.

## Cuándo importa

Arquitecturas de microservicios, entornos multi-tenant, equipos remotos donde ya no existe un único perímetro de oficina que proteger. Es la base conceptual detrás de productos como BeyondCorp (Google) o de que un service mesh haga mTLS por default entre servicios — la idea de que la ubicación en la red dejó de ser una señal de confianza válida.

---
Relacionado: [Autenticación y Seguridad](../backend/autenticacion.md), [SQL Injection](sql-injection.md#cómo-se-previene), [Riesgos y Mitigaciones en Agentes de IA](../agentic-ai/riesgos-y-mitigaciones.md#mitigaciones-técnicas).
