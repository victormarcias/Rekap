# Redis

Key-Value store **en memoria**, extremadamente rápido (operaciones en microsegundos) — mucho más que cualquier DB en disco. El [Cache-aside pattern](../diagnostico/backend.md#falta-de-cache) ya lo usó como ejemplo; acá está la herramienta en sí y sus otros usos más allá de cache.

## No es solo cache

Redis persiste opcionalmente a disco (snapshots RDB periódicos, o un log de escritura AOF) — no es memoria puramente volátil como a veces se asume. Eso habilita usarlo para más que cache descartable:

- **Cache**: el uso más común — ver [Cache-aside](../diagnostico/backend.md#falta-de-cache) y [Cache invalidation](../backend/cache-invalidation.md).
- **Session store**: guardar sesiones de usuario en un store compartido, no en memoria del proceso — la base de por qué [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad) horizontal necesita esto.
- **Rate limiting**: un contador con TTL por cliente/IP es trivial en Redis (`INCR` + `EXPIRE`), y es atómico incluso con múltiples instancias del backend pegándole al mismo Redis.
- **Pub/Sub**: canales de mensajería simple — no tiene la durabilidad ni el replay de [Kafka](../backend/kafka.md), pero es liviano para notificaciones en tiempo real que no necesitan sobrevivir un restart.
- **Distributed lock**: coordinar que solo un proceso (de varios corriendo en paralelo) haga cierta tarea a la vez.

```python
# rate limiting: contador atómico con expiración — 100 requests por minuto por usuario
def is_rate_limited(user_id):
    key = f"ratelimit:{user_id}"
    count = redis.incr(key)          # atómico: incrementa y devuelve el nuevo valor
    if count == 1:
        redis.expire(key, 60)         # arranca el TTL en el primer request de la ventana
    return count > 100
```

## Más que strings

A diferencia de un cache simple clave→string, Redis tiene estructuras de datos nativas — listas, sets, sorted sets (útiles para leaderboards/rankings, ordenados por score), hashes (un objeto con campos, sin serializar/deserializar todo el blob para tocar un solo campo).

```python
# sorted set: ranking de usuarios por puntaje, siempre ordenado, sin resortear a mano
redis.zadd("leaderboard", {"user_42": 1500})
redis.zrevrange("leaderboard", 0, 9)  # top 10, ya ordenado
```

## El trade-off

Todo vive en RAM — rápido, pero el dataset está limitado por la memoria disponible (a diferencia de una DB en disco), y si Redis se cae sin persistencia configurada, se pierde todo lo que no llegó a un snapshot. No reemplaza a la base de datos principal — es un complemento para lo que necesita velocidad extrema o estructuras específicas (contadores, rankings, colas cortas).

---
Relacionado: [Cache invalidation](../backend/cache-invalidation.md), [Diagnóstico Backend](../diagnostico/backend.md#falta-de-cache), [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad).
