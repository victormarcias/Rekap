# REST

Estilo arquitectónico para diseñar APIs — no es lo mismo que "usar HTTP methods" (eso es solo una de sus seis restricciones).

## Las 6 restricciones

1. **Cliente-servidor**: separación de responsabilidades — el cliente no sabe cómo está implementado el servidor (lenguaje, DB), el servidor no sabe cómo se renderiza la UI. Cada lado evoluciona independiente.
2. **Stateless**: cada request trae toda la información necesaria para procesarlo — el servidor no guarda contexto de requests anteriores del mismo cliente entre llamadas. Ver [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad) (sesión compartida vs sesión en memoria — statelessness es lo que lo habilita).
3. **Cacheable**: las respuestas indican explícitamente si son cacheables (`Cache-Control`), para que el cliente o intermediarios las reusen sin volver a pedirlas. Ver [CDN](../devops/cdn.md).
4. **Interfaz uniforme**: recursos identificados por URLs, manipulados con verbos HTTP estándar (ver [HTTP Methods](http-methods.md)) y representaciones auto-descriptivas (JSON con Content-Type) — más HATEOAS, ver abajo.
5. **Sistema en capas**: el cliente no puede (ni necesita) saber si habla directo con el servidor de origen o con un proxy/gateway/load balancer en el medio. Ver [API Gateway](api-gateway.md), [Load balancers](load-balancers.md).
6. **Code-on-demand** (opcional): el servidor puede extender la funcionalidad del cliente mandando código ejecutable — la única restricción no obligatoria de las seis.

## Convenciones de recursos (parte de la interfaz uniforme)

```
❌ POST /createOrder
✅ POST /orders

❌ GET /getOrderItems?orderId=1
✅ GET /orders/1/items
```

Sustantivos plurales en vez de verbos en la URL (el verbo ya lo dice el método HTTP), y anidamiento para expresar relaciones entre recursos.

## HATEOAS — la restricción que casi nadie implementa

**Hypermedia As The Engine Of Application State**: cada respuesta debería incluir links a las acciones/recursos disponibles desde ese estado, para que el cliente navegue la API dinámicamente en vez de tener URLs hardcodeadas de antemano — como un browser siguiendo links en un sitio web, en vez de tener memorizada de antemano la estructura entera.

```json
{
  "id": 1,
  "status": "pending",
  "total": 100,
  "_links": {
    "self": { "href": "/orders/1" },
    "cancel": { "href": "/orders/1/cancel", "method": "POST" },
    "pay": { "href": "/orders/1/pay", "method": "POST" }
  }
}
```

Sin esto, el cliente necesita conocer de antemano **todas** las URLs posibles, típicamente por documentación externa — con HATEOAS, la API es auto-descubrible.

## "RESTful" vs REST real

En la práctica, casi ninguna API que se autodenomina "REST" implementa HATEOAS — la mayoría cumple solo con "recursos + verbos HTTP + JSON", que es apenas la mitad de la restricción de interfaz uniforme. Vale la pena tenerlo claro: si te preguntan "¿tu API es REST?", la respuesta honesta casi siempre es "es HTTP con convenciones REST, no REST completo".

---
Relacionado: [HTTP Methods](http-methods.md), [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad), [API Gateway](api-gateway.md), [CDN](../devops/cdn.md), [GraphQL](graphql.md) (la alternativa).
