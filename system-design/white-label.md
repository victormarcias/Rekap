# White-Label: Consideraciones para Construir una App White-Label

**Qué es**: un mismo producto que se vende/licencia a múltiples clientes, y cada uno lo presenta como si fuera su propia marca (logo, colores, dominio) — el usuario final del cliente no sabe, ni le importa, que por debajo corre la misma plataforma que usan otros clientes del mismo proveedor. La diferencia con "una app configurable" es de fondo: acá **una sola base de código e infraestructura** tiene que servir a N clientes distintos, sin que se mezclen entre sí.

## Multi-tenancy: el problema arquitectónico central

Cada cliente es un **tenant**. La pregunta de diseño de fondo: cómo separar los datos y la configuración de cada tenant, corriendo todos sobre la misma infraestructura.

### Estrategias de aislamiento de datos

| Estrategia | Aislamiento | Costo operativo | Complejidad de queries |
|---|---|---|---|
| **DB separada por tenant** | Alto — un tenant no puede tocar datos de otro ni por accidente | Alto — N bases de datos que mantener, migrar, backupear | Baja — cada query ya está scoped por naturaleza |
| **Schema separado, DB compartida** | Medio | Medio | Baja-media |
| **Fila compartida (`tenant_id` en cada tabla)** | Bajo — depende 100% de que el código filtre bien | Bajo — una sola DB para todos | Alta — cada query necesita el filtro correcto |

```sql
-- patrón más común (shared DB, shared schema): tenant_id en cada tabla
SELECT * FROM pedidos WHERE tenant_id = 'cliente-123' AND id = 42;

-- olvidar el WHERE tenant_id es el bug más caro que existe en un sistema white-label:
-- un cliente termina viendo (o peor, editando) los datos de otro
```

Es el mismo problema de fondo que [Sharding vs Partitioning](../database/sharding-vs-partitioning.md) (separar datos por alguna clave), aplicado a nivel de cliente en vez de a nivel de volumen de datos.

## Branding dinámico: theming sin tocar código

Cada tenant necesita su logo, paleta de colores, tipografía — sin que eso implique un deploy o una rama de código por cliente. El patrón estándar: **CSS Variables** (ver [CSS](../frontend-react/css.md#css-variables-custom-properties)) cargadas en runtime según el tenant activo, más un objeto de config con las URLs de sus assets.

```json
// config de tenant, resuelta según el dominio/subdominio del request
{
  "tenantId": "cliente-123",
  "branding": {
    "primaryColor": "#1a73e8",
    "logoUrl": "https://cdn.miapp.com/tenants/cliente-123/logo.svg",
    "appName": "Acme Dashboard"
  }
}
```

## Dominios: subdominio vs dominio propio

- **Subdominio** (`cliente1.miapp.com`): simple — un solo certificado TLS wildcard sirve a todos, el código lee el subdominio del request para saber qué tenant es.
- **Dominio propio del cliente** (`app.clienteweb.com`, con un CNAME apuntando a tu infra): más profesional (el cliente no ve tu marca en la URL), pero cada dominio necesita su propio certificado ([TLS handshake](que-pasa-cuando-escribis-una-url.md#4-tls-handshake-si-es-https)) — Let's Encrypt automatiza la emisión/renovación, pero es infraestructura extra a mantener y a monitorear.

## Seguridad: el aislamiento entre tenants no es opcional

En un producto white-label, un bug que filtra datos de un tenant a otro no es un bug cualquiera — es el peor escenario posible (un cliente viendo datos de otro cliente, a veces su competencia directa). Esto empuja hacia:

- **Middleware que inyecta el `tenant_id` automáticamente** en cada query, en vez de confiar en que cada desarrollador se acuerde de agregarlo a mano.
- **Tests automatizados específicos de aislamiento**: crear 2 tenants de prueba y verificar que ningún query de uno devuelve datos del otro.
- El tenant como parte del claim del [JWT](../backend/authentication.es.md#5-jwt--estructura-y-stateless) — algo que se valida en cada request, no algo que se infiere después de autenticar.

## Feature flags y planes por tenant

No todos los tenants necesitan las mismas funcionalidades — un producto white-label típicamente vende distintos planes (básico/pro/enterprise), cada uno con un set de features habilitadas. Se maneja con feature flags evaluados contra la config del tenant activo — **nunca con ramas de código distintas por cliente**, mantener N forks del mismo producto no escala.

## Onboarding: crear un tenant nuevo es un proceso, no un deploy

Si agregar un cliente nuevo implica que alguien del equipo entre a tocar código o infraestructura a mano, el modelo no escala. La meta es que "crear un tenant" sea una operación automatizada y repetible: correr una migración que crea su registro, generar su config de branding, aprovisionar su subdominio — sin intervención manual en cada alta.

## Por qué importa

Es el mismo problema que resuelve cualquier sistema multi-cliente a escala (SaaS B2B, plataformas para agencias) — la arquitectura correcta desde el día uno evita una migración dolorosa después: pasar de "todo compartido sin `tenant_id`" a un modelo con aislamiento real, con datos ya mezclados en producción, sale mucho más caro que diseñarlo bien desde el principio.

---
Relacionado: [Sharding vs Partitioning](../database/sharding-vs-partitioning.md), [CSS Variables](../frontend-react/css.md#css-variables-custom-properties), [Autenticación y Seguridad](../backend/authentication.es.md), [Atributos de calidad de sistemas](atributos-de-calidad.md), [Qué pasa cuando escribís una URL](que-pasa-cuando-escribis-una-url.md).
