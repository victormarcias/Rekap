# DDoS Attack

**Distributed Denial of Service**: múltiples fuentes (a menudo una botnet — miles de máquinas comprometidas) inundan un servicio con tráfico, buscando agotar sus recursos (CPU, red, conexiones) hasta dejarlo inaccesible para usuarios legítimos. "Distribuido" es la palabra clave — a diferencia de un DoS de una sola fuente, no alcanza con bloquear una IP.

## Tipos, de más simple a más difícil de mitigar

- **Volumétrico**: satura directamente el ancho de banda disponible con un volumen masivo de tráfico — no importa si las requests son válidas, el enlace de red se llena antes de que lleguen a procesarse.
- **De protocolo**: explota el comportamiento de un protocolo en vez del ancho de banda. El clásico es el **SYN flood**: mandar miles de `SYN` (el primer paso del [three-way handshake](../system-design/que-pasa-cuando-escribis-una-url.md#3-conexión-tcp--three-way-handshake)) sin nunca completar el `ACK` final — el servidor reserva recursos para cada conexión "medio abierta" hasta agotarlos.
- **De aplicación**: requests HTTP completamente legítimas en apariencia, pero en un volumen que satura la capacidad de procesamiento del backend — el más difícil de distinguir de un pico de tráfico real, porque cada request individual parece normal.

## Mitigaciones por capa

- **CDN/edge**: absorbe el tráfico volumétrico antes de que llegue al origin — ver [CDN](../devops/cdn.es.md). La mayoría del tráfico de un ataque grande ni siquiera llega a tocar tu infraestructura real.
- **Rate limiting**: acota cuántos requests acepta por IP/cliente en una ventana de tiempo, devolviendo [429 Too Many Requests](../system-design/http-status-codes.md#4xx--client-error) al resto — normalmente centralizado en el [API Gateway](api-gateway.es.md), no en cada servicio.
- **WAF (Web Application Firewall)**: filtra patrones de tráfico maliciosos conocidos antes de que lleguen a la aplicación.
- **Autoscaling — con cuidado**: escalar automáticamente ante un pico de tráfico ayuda a absorber el ataque, pero sin un techo puede convertirse en un **"ataque de denegación de servicio económico"**: el atacante no tira el servicio, pero factura una cuenta de cloud enorme escalando infraestructura que nunca sirvió tráfico real. Ver [Elasticidad](../system-design/atributos-de-calidad.md#elasticidad) — siempre con un `maxReplicas` (o equivalente) como límite duro.

## La parte difícil: distinguir ataque de tráfico legítimo

Un contenido que se vuelve viral genera un patrón de tráfico parecido a un ataque de aplicación — mucho volumen, de muchas fuentes distintas, todo pidiendo lo mismo. La diferencia suele estar en el detalle: un ataque tiende a tener patrones más uniformes/artificiales (mismo user-agent, mismos intervalos, geografías inusuales para el negocio) que un pico orgánico real — pero la línea no siempre es nítida, y por eso el rate limiting y el WAF trabajan con heurísticas, no con certezas.

---
Relacionado: [CDN](../devops/cdn.es.md), [API Gateway](api-gateway.es.md), [Qué pasa cuando escribís una URL](../system-design/que-pasa-cuando-escribis-una-url.md) (TCP handshake), [Elasticidad](../system-design/atributos-de-calidad.md#elasticidad), [Security](../security/) (vulnerabilidades de aplicación, distinto de un ataque de tráfico).
