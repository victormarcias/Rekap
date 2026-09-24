# Monolito vs Microservicios

Dos formas de organizar el código y el deploy de un sistema backend — no es "microservicios siempre gana", es un trade-off real.

## Monolito

Toda la aplicación es **un solo codebase, un solo proceso, un solo deploy**. Las distintas partes (órdenes, usuarios, pagos) se comunican entre sí con llamadas de función directas, en memoria — no hay red de por medio.

**A favor**: simple de desarrollar, testear y deployar al principio; una transacción de DB que cruza "módulos" es una transacción normal (ver [ACID](../database/acid-transacciones-isolation.md)); no hay latencia de red entre componentes internos.

**En contra**: escalar significa escalar **todo** el proceso aunque solo una parte tenga mucha carga; un bug en un módulo puede tirar abajo toda la app; a medida que el equipo crece, todos pisándose el mismo repo/deploy se vuelve un cuello de botella organizacional, no técnico.

## Microservicios

Cada parte del sistema es un **servicio independiente**, con su propio proceso, su propio deploy, y (idealmente) su propia base de datos. Se comunican por red — HTTP, o de forma asíncrona vía colas/eventos (ver [Arquitectura Kafka](kafka.es.md), [Colas de mensajes](message-queues.es.md)).

**A favor**: cada servicio escala de forma independiente (ver [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad)); un equipo puede deployar su servicio sin coordinar con los demás; un bug en un servicio no necesariamente tira abajo los otros.

**En contra**: lo que antes era una llamada de función ahora es una llamada de red — más lenta, y puede fallar (ver [Tolerancia a fallos](../system-design/atributos-de-calidad.md#tolerancia-a-fallos)); una transacción que cruza dos servicios ya no es una transacción de DB simple — hay que resolverlo con patrones más complejos (sagas, eventual consistency); mucha más complejidad operacional (monitoreo, deploys, versionado de contratos entre servicios).

## El anti-patrón: distributed monolith

El peor de los dos mundos: servicios separados en el deploy, pero tan acoplados entre sí (llamadas síncronas encadenadas, schemas de DB compartidos, deploys que tienen que coordinarse en un orden específico) que en la práctica **hay que deployarlos todos juntos** para que algo funcione. Se pagan todos los costos de microservicios (latencia de red, complejidad operacional) sin ninguno de los beneficios (independencia real).

## Regla práctica

"Monolito first" es un consejo común: arrancar con un monolito bien organizado por capas (ver [Controller / Service / Repository](controller-service-repository.es.md)) y **extraer** servicios recién cuando una parte específica realmente necesita escalar o deployarse de forma independiente — no partir en microservicios desde el día uno sin tener todavía claro dónde están los límites naturales del dominio.

---
Relacionado: [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad), [Controller / Service / Repository](controller-service-repository.es.md), [API Gateway](api-gateway.es.md).
