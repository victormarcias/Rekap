# Atributos de Calidad de Sistemas ("-ilities")

Propiedades no funcionales que definen qué tan bueno es un diseño más allá de "funciona" — la mayoría de las decisiones de arquitectura son tradeoffs conscientes entre estos atributos, no la maximización de uno solo.

## Escalabilidad

Capacidad de un sistema de manejar más carga (usuarios, requests, datos) sumando recursos, sin degradar performance ni requerir un rediseño. El bloqueo más común: guardar estado en memoria del proceso, lo que ata un usuario a una instancia específica e impide escalar horizontalmente sin problemas.

```js
// ❌ sesión en memoria del proceso: no escala horizontalmente —
// el siguiente request de ese usuario puede caer en otra instancia que no tiene esta sesión
const sessions = {};
app.post('/login', (req, res) => { sessions[req.body.userId] = { loggedIn: true }; });

// ✅ estado en un store externo compartido (Redis): cualquier instancia puede atenderlo
app.post('/login', async (req, res) => {
  await redis.set(`session:${req.body.userId}`, JSON.stringify({ loggedIn: true }));
});
```

Ver [Escalabilidad vertical vs horizontal](../diagnostics/devops.es.md) para la implementación concreta a nivel infraestructura, y [Sharding vs partitioning](../database/sharding-vs-partitioning.md) para el ángulo de base de datos.

## Inmutabilidad

Una vez creado un dato, no cambia — cualquier "modificación" produce una copia nueva. Importa porque elimina una clase entera de bugs de concurrencia (nadie puede mutar algo que otro está leyendo al mismo tiempo — ver [Locks](../database/locks.md)), hace el estado predecible, y permite detectar cambios comparando referencias en vez de hacer una comparación profunda costosa — es la base de cómo React decide si re-renderizar.

```python
# ❌ mutable: cualquier código con una referencia al objeto puede alterarlo
# sin que el resto de la app se entere, generando efectos secundarios ocultos
config = {"retries": 3}
def process(cfg):
    cfg["retries"] = 0

# ✅ inmutable: cualquier cambio genera un objeto nuevo, el original queda intacto
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class Config:
    retries: int = 3

new_config = replace(config, retries=0)  # config original no cambió
```

```ts
// ❌ mutar el estado directo: React compara por referencia y no detecta el cambio (mismo objeto)
const [user, setUser] = useState({ name: 'Ana', age: 30 });
user.age = 31; // no dispara re-render

// ✅ crear un objeto nuevo: la referencia cambia, React sí re-renderiza
setUser({ ...user, age: 31 });
```

### Shallow copy vs Deep copy

El spread (`{...obj}`), `Object.assign`, `dict.copy()` o `dataclasses.replace` de los ejemplos de arriba solo copian el **primer nivel**. Cualquier objeto o array anidado adentro sigue siendo la **misma referencia** que en el original — mutar ese nivel interno rompe la garantía de inmutabilidad sin que se note, porque el nivel superior sí parece "nuevo".

```python
import copy

original = {"user": {"name": "Ana"}, "count": 1}

shallow = original.copy()          # o dict(original), o {**original}
shallow["count"] = 2                # ✅ no afecta a original — el nivel superior sí se copió
shallow["user"]["name"] = "Beto"     # ❌ esto SÍ modifica original["user"]["name"] — misma referencia anidada

deep = copy.deepcopy(original)
deep["user"]["name"] = "Carla"        # ✅ ahora no toca el original en ningún nivel
```

```ts
const original = { user: { name: 'Ana' }, count: 1 };

const shallow = { ...original };
shallow.count = 2;              // ✅ no afecta a original
shallow.user.name = 'Beto';      // ❌ sí afecta a original.user.name — misma referencia anidada

const deep = structuredClone(original); // copia profunda nativa (o JSON.parse(JSON.stringify(x)) como alternativa vieja)
deep.user.name = 'Carla';         // ✅ no toca el original en ningún nivel
```

Por eso `setUser({ ...user, age: 31 })` de arriba es seguro — `age` es un valor primitivo, no un objeto anidado. Si `user` tuviera `address: { city: 'BA' }`, ese mismo spread superficial no alcanzaría para modificar `city` sin mutar el original: haría falta `{ ...user, address: { ...user.address, city: 'Rosario' } }` (o una librería de manejo de estado inmutable) para sostener la garantía en niveles más profundos.

## Disponibilidad

Proporción del tiempo que el sistema responde correctamente. Se mide en "nueves" (99.9% ≈ 8.7 horas de downtime al año, 99.99% ≈ 52 minutos). Se logra con redundancia: más de una instancia corriendo, en más de una zona de disponibilidad, con un load balancer que deja de mandar tráfico a la que falla.

```yaml
# ✅ K8s: sin esto, un pod caído sigue recibiendo tráfico hasta que alguien lo note
readinessProbe:
  httpGet: { path: /health, port: 3000 }
  periodSeconds: 10
```

Trade-off directo con **Consistencia** (ver más abajo) — es la esencia del CAP theorem, ver [NoSQL](../database/nosql.md).

## Consistencia

Todos los nodos/lectores del sistema ven el mismo dato al mismo tiempo. **Consistencia fuerte**: una escritura es visible inmediatamente para toda lectura posterior (más simple de razonar, más caro de lograr en sistemas distribuidos). **Consistencia eventual**: una escritura tarda un rato en propagarse a todos los nodos, pero converge (más disponible, más barato, pero un usuario puede ver datos viejos por un momento).

```sql
-- ✅ Postgres, consistencia fuerte dentro de una transacción:
-- ninguna otra conexión ve el balance actualizado hasta el COMMIT
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

Ver [ACID / isolation levels](../database/acid-transacciones-isolation.md) para consistencia fuerte, y [NoSQL](../database/nosql.md) para el trade-off con disponibilidad (CAP theorem).

## Tolerancia a fallos

El sistema sigue funcionando (aunque sea en modo degradado) cuando falla un componente, en vez de caerse en cascada. El patrón más común: **circuit breaker** — deja de intentar contra un servicio que ya demostró estar caído, en vez de acumular timeouts que agotan recursos propios.

```ts
// ✅ circuit breaker simplificado: corta los intentos tras fallos repetidos
class CircuitBreaker {
  private failures = 0;
  private open = false;

  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.open) throw new Error('Circuit open — servicio no disponible, no reintentamos');
    try {
      const result = await fn();
      this.failures = 0; // se recupera tras un éxito
      return result;
    } catch (err) {
      if (++this.failures >= 3) this.open = true; // corta tras 3 fallos seguidos
      throw err;
    }
  }
}
```

## Idempotencia

Ejecutar la misma operación varias veces produce el mismo resultado que ejecutarla una sola vez. Crítico para reintentos seguros ante fallos de red: si un cliente manda un request, se cae la conexión antes de recibir la respuesta, y reintenta — ¿el servidor ya procesó el primer intento o no? Sin idempotencia, ese reintento puede duplicar un cobro.

```python
# ❌ no idempotente: reintentar tras un timeout puede cobrar dos veces
def charge(amount, card):
    return payment_gateway.create_charge(amount, card)

# ✅ idempotente: la misma idempotency_key nunca genera un segundo cobro,
# el servidor la usa para detectar "ya procesé este request antes"
def charge(amount, card, idempotency_key):
    return payment_gateway.create_charge(amount, card, idempotency_key=idempotency_key)
```

En HTTP, `GET`/`PUT`/`DELETE` están definidos como idempotentes por spec; `POST` no — por eso `POST` es el verbo más riesgoso para reintentar automáticamente sin una idempotency key.

## Observabilidad

Qué tan bien se puede entender el estado interno de un sistema mirando sus outputs externos (logs, métricas, traces), sin tener que modificarlo o debuggear en vivo. El pilar que más se olvida: sin un **correlation ID** que viaje entre servicios, un log individual no sirve para reconstruir qué pasó con un request que cruzó 5 microservicios.

```js
// ✅ log estructurado con correlation id — permite rastrear un request across servicios
logger.info({ event: 'order_created', orderId, correlationId: req.headers['x-correlation-id'] });
```

## Elasticidad

Caso particular de escalabilidad: la capacidad de escalar automáticamente hacia arriba **y hacia abajo** según la demanda en tiempo real, sin intervención manual. La diferencia con "escalabilidad" a secas es esa automatización — un sistema puede ser escalable (soporta más carga si le agregás recursos a mano) sin ser elástico (nadie se los agrega solo).

```yaml
# ✅ K8s HPA: agrega/quita pods solo, según el uso real de CPU
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 70 } }
```

## Mantenibilidad

Qué tan fácil es entender, modificar y extender el sistema sin romper cosas ni tardar cada vez más en cada cambio. No hay un "medidor" directo — se infiere de proxies: código acoplado vs desacoplado, tests que dan confianza para refactorizar sin miedo, una misma regla de negocio viviendo en un solo lugar en vez de copiada en varios.

```python
# ❌ el mismo "threshold mágico" repetido en varios lugares —
# cambiarlo implica encontrar y actualizar cada aparición, con riesgo de olvidarse una
def can_checkout(cart_total):
    return cart_total >= 50

def apply_free_shipping(cart_total):
    return cart_total >= 50

# ✅ una sola fuente de verdad — cambiar la regla de negocio es un solo lugar
FREE_SHIPPING_THRESHOLD = 50

def can_checkout(cart_total):
    return cart_total >= FREE_SHIPPING_THRESHOLD
```

[SOLID](solid.md) es, en esencia, un conjunto de principios pensados para maximizar esto — el Open/Closed Principle en particular ("abierto a extensión, cerrado a modificación") es la versión más directa de "agregar una feature nueva sin tener que tocar código que ya funciona y ya está probado".

## Performance (Velocidad)

Qué tan rápido responde el sistema — dos caras que no siempre van juntas: **latencia** (cuánto tarda una operación individual) y **throughput** (cuántas operaciones por segundo puede sostener). Optimizar una a veces empeora la otra (ej. procesar en batches grandes mejora throughput pero empeora la latencia de cada request individual que espera a que se junte el batch).

```
p50 (mediana): 90ms   — la mitad de los requests responde más rápido que esto
p95: 320ms
p99: 1800ms            — 1 de cada 100 requests tarda esto o más

El promedio (ej. 150ms) esconde el p99: si un 1% de tus usuarios
sufre 1.8 segundos de latencia, el promedio solo no lo muestra —
mirar percentiles altos es la única forma de ver esa cola larga.
```

Medirlo bien importa más que cualquier técnica puntual — ver [Performance Diagnostics](../frontend-react/performance-diagnostics.md) y [Web Vitals](../frontend-react/web-vitals.md) del lado frontend, [Query Optimization](../database/query-optimization.md) del lado de base de datos, y [Diagnóstico Backend](../diagnostics/backend.es.md) / [Diagnóstico Frontend](../diagnostics/frontend.es.md) para las causas más comunes de un sistema lento.

## Resumen

| Atributo | En una frase |
|---|---|
| Escalabilidad | Aguanta más carga sumando recursos |
| Inmutabilidad | Los datos no cambian, se reemplazan |
| Disponibilidad | El sistema responde cuando lo necesitás |
| Consistencia | Todos ven el mismo dato al mismo tiempo |
| Tolerancia a fallos | Sigue funcionando aunque algo se rompa |
| Idempotencia | Repetir la operación no cambia el resultado |
| Observabilidad | Podés entender qué pasa adentro desde afuera |
| Elasticidad | Escala sola, arriba y abajo, según demanda |
| Mantenibilidad | Fácil de modificar sin romper todo lo demás |
| Performance | Responde rápido y sostiene la carga |
| Seguridad | Protege datos y accesos — ver [Autenticación y Seguridad](../backend/authentication.es.md) |

Estos atributos suelen tironear entre sí (el ejemplo clásico: más Consistencia generalmente cuesta Disponibilidad). Toda decisión de arquitectura es elegir conscientemente qué priorizar para el caso de uso — no existe un diseño que maximice todos a la vez.
