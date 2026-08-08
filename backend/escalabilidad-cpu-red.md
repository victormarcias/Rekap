# Escalabilidad de CPU y Red

[Escalabilidad vertical vs horizontal](../devops/escalabilidad-vertical-horizontal.md) ya cubre el **cómo** escalar. Esto es el paso previo: identificar **cuál** es el recurso que realmente se está agotando, porque la solución es distinta según sea CPU o red.

## Cuello de botella de CPU

El proceso pasa la mayor parte del tiempo **calculando** — serialización pesada, criptografía, procesamiento de imágenes, loops sobre datasets grandes. Se nota con el uso de CPU cerca del 100% mientras el resto de los recursos (memoria, red) están relajados.

**Diagnóstico**: un profiler del runtime (`py-spy`, `node --prof`) muestra en qué función se está gastando el tiempo de CPU real.

**Soluciones**: paralelizar el cómputo (workers, `multiprocessing`), optimizar el algoritmo antes que la infraestructura (ver [Algoritmos no optimizados](../diagnostico/backend.md#algoritmos-no-optimizados)), o escalar verticalmente (más cores) si ya está bien paralelizado.

## Cuello de botella de red

El proceso pasa la mayor parte del tiempo **esperando** — no porque no tenga qué hacer, sino porque está bloqueado esperando una respuesta de otro servicio, o porque el ancho de banda disponible ya está saturado. Se nota con CPU relajada pero latencia alta y throughput bajo.

Causas típicas:
- **Chattiness**: muchas llamadas de red chicas en vez de pocas grandes — cada llamada paga su propio round trip (ver [HTTP chaining](../diagnostico/backend.md#http-chaining)).
- **Payloads grandes sin comprimir**: mandar JSON completo sin `gzip`/`br` cuando el cliente lo soporta.
- **Ancho de banda saturado**: el enlace entre servicios (o hacia afuera) ya está al límite de lo que puede transmitir, sin importar cuánta CPU sobre.

```python
# ❌ chatty: 3 round trips de red para lo que podría ser 1
user = await fetch_user(id)
orders = await fetch_orders(id)
reviews = await fetch_reviews(id)

# ✅ batchear: 1 round trip, o al menos en paralelo (ver HTTP chaining en diagnóstico backend)
user, orders, reviews = await asyncio.gather(fetch_user(id), fetch_orders(id), fetch_reviews(id))
```

**Soluciones**: batchear/paralelizar llamadas, comprimir respuestas, [connection pooling](../diagnostico/backend.md#conexiones-mal-gestionadas), acercar los datos al consumidor con una [CDN](../devops/cdn.md), o reducir la cantidad de saltos de red rediseñando los límites entre servicios (ver [Monolito vs Microservicios](monolito-vs-microservicios.md) — más microservicios generalmente significa más tráfico de red interno).

## Por qué distinguirlos importa

Escalar verticalmente (más CPU) no ayuda en nada si el cuello de botella es la red — el proceso va a seguir esperando igual, solo que con más cores ociosos. Diagnosticar mal el tipo de cuello de botella lleva a "soluciones" caras que no tocan el problema real.

---
Relacionado: [Diagnóstico Backend](../diagnostico/backend.md), [Escalabilidad vertical vs horizontal](../devops/escalabilidad-vertical-horizontal.md), [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad).
