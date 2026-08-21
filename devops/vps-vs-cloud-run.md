# VPS vs Cloud Run

## Qué es un VPS

**Virtual Private Server**: una porción de un servidor físico grande, virtualizada para que se comporte como una máquina propia e independiente — acceso root, tu propio SO, recursos (CPU/RAM/disco) garantizados y aislados del resto, aunque el hardware físico de abajo se comparta con otros VPS del mismo servidor (ver [Virtualización](../devops/docker.md#virtualización--el-origen-de-los-contenedores) para el mecanismo técnico). Se diferencia de:

- **Shared hosting**: ahí ni siquiera tenés tu propio SO — compartís el mismo proceso/entorno con otros clientes, sin acceso root.
- **Servidor dedicado**: hardware físico completo para vos solo, sin virtualización — más caro, sin los límites que impone compartir hardware.

Proveedores típicos: DigitalOcean, Linode/Akamai, Hetzner, AWS EC2.

Comparación entre [Deploy a un VPS](deploy-vps.md) (servidor propio, siempre prendido) y [Deploy a Cloud Run](deploy-cloud-run.md) (contenedor serverless, escala a cero).

## El espectro completo: IaaS → PaaS → Serverless

VPS y Cloud Run son dos puntos de un espectro más amplio — a medida que avanzás, le delegás más responsabilidad a la plataforma, a cambio de menos control:

- **IaaS** (Infrastructure as a Service): un VPS es esto — te dan una máquina virtual vacía, vos instalás el SO, el runtime, todo. Máximo control, máxima responsabilidad.
- **PaaS** (Platform as a Service): le das tu código (o un `git push`), la plataforma se encarga del SO, el runtime y el proceso de deploy — ya no tocás un servidor directamente. Ejemplos: Heroku, Render, Railway. Es un paso intermedio entre VPS y serverless: no hay cold start ni escala a cero (normalmente corre siempre, como el VPS), pero tampoco administrás infraestructura.
- **Serverless / CaaS** (Container as a Service): Cloud Run es esto — le das un contenedor, la plataforma decide cuántas instancias correr y cuándo, incluyendo escalar a cero. Cero infraestructura que administrar, pero con cold starts y menos control fino.
- **FaaS** (Function as a Service, ej. AWS Lambda): un paso más allá de Cloud Run — ni siquiera un contenedor, solo una función individual que la plataforma ejecuta bajo demanda.

**Dónde entra PaaS en la práctica**: es la opción típica para no lidiar con servidores sin pagar el costo de cold starts — a cambio, suele ser más caro que un VPS equivalente a tráfico alto y constante (mismo trade-off que Cloud Run, pero sin el beneficio de escalar a cero).

## Comparación técnica

| | VPS | Cloud Run |
|---|---|---|
| Quién administra el SO | Vos (parches, firewall, SSH) | Google |
| Costo con cero tráfico | Full precio, 24/7 | ~$0 (escala a cero) |
| Cold start | No existe (siempre prendido) | Sí, en la primera request tras estar en cero |
| Certificado HTTPS | Manual (`certbot`, renovación cada 90 días) | Automático |
| Control fino de infraestructura | Total | Limitado a lo que expone la plataforma |

## Pros y contras

| | VPS | Cloud Run |
|---|---|---|
| **Pros** | Costo fijo y predecible con tráfico alto y constante; control total (podés instalar lo que sea, tunear el kernel, correr varios servicios en la misma máquina); sin cold start; sin atarte a un proveedor específico | Cero mantenimiento de SO/parches; escala automáticamente con la demanda (y a cero cuando no hay tráfico); HTTPS y balanceo de carga resueltos por la plataforma; pagás por uso real, no por capacidad reservada |
| **Contras** | Vos sos responsable de seguridad, updates y capacity planning; pagás lo mismo tenga tráfico o no; escalar significa aprovisionar más máquinas a mano (o armar tu propio autoscaling) | Cold start en tráfico esporádico; menos control (no hay acceso al SO subyacente); a tráfico alto y sostenido puede terminar costando más que una máquina propia; atado a las convenciones de la plataforma (límites de tiempo de request, tamaño de imagen, etc.) |

---
Relacionado: [Deploy a un VPS](deploy-vps.md), [Deploy a Cloud Run](deploy-cloud-run.md), [Cold start](../diagnostico/devops.md), [Elasticidad](../system-design/atributos-de-calidad.md#elasticidad).
