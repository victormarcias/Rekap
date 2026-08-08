# API Gateway

Un único punto de entrada que recibe todo el tráfico externo y lo rutea hacia el servicio interno correcto — se confunde seguido con un Load Balancer, pero resuelven problemas distintos.

## Gateway vs Load Balancer

Un [Load Balancer](load-balancers.md) reparte tráfico entre **réplicas del mismo servicio** (round robin, least connections). Un API Gateway rutea entre **servicios distintos** (`/orders` va al servicio de órdenes, `/users` va al servicio de usuarios) y además suele resolver responsabilidades transversales que ningún servicio individual debería tener que implementar por su cuenta:

- **Autenticación centralizada**: valida el token una sola vez, en el borde, antes de que el request llegue a ningún servicio interno.
- **Rate limiting**: aplica límites de uso por cliente/API key en un solo lugar (ver [429 Too Many Requests](../system-design/http-status-codes.md#4xx--client-error)).
- **Transformación de request/response**: adapta formatos entre lo que expone el cliente externo y lo que espera cada servicio interno.
- **Agregación**: un solo request del cliente puede traducirse en varias llamadas a distintos servicios internos, combinando las respuestas en una sola — el patrón **BFF (Backend for Frontend)** es una variante de esto, un gateway a medida para cada tipo de cliente (web, mobile).

```
Cliente → API Gateway → /orders/*  → Order Service
                       → /users/*   → User Service
                       → /payments/* → Payment Service
```

## Por qué importa en microservicios

Sin gateway, cada [microservicio](monolito-vs-microservicios.md) tendría que implementar su propia autenticación, su propio rate limiting, su propio manejo de CORS — repetido N veces, con N oportunidades de hacerlo distinto o de tener un bug de seguridad en uno solo. El gateway centraliza eso una sola vez, en el borde del sistema.

**El costo**: es un punto único de falla si no está bien redundado, y agrega un salto de red extra (latencia) a cada request.

## Herramientas comunes

Kong, AWS API Gateway, Traefik (que ya mencionamos como L7 en [Load balancers](load-balancers.md) — muchas herramientas hacen tanto de load balancer como de gateway liviano, la línea entre ambos roles no siempre es nítida en la práctica).

---
Relacionado: [Load balancers](load-balancers.md), [Monolito vs Microservicios](monolito-vs-microservicios.md), [Autenticación y Seguridad](autenticacion.md).
