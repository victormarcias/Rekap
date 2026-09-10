# AWS — Servicios Principales

Mapa rápido de los servicios más nombrados de AWS y para qué sirve cada uno — varios ya tienen su propio desarrollo en otras carpetas del repo (se linkean en vez de repetir).

| Servicio | Categoría | Caso de uso típico |
|---|---|---|
| **EC2** (Elastic Compute Cloud) | Cómputo | Máquina virtual de propósito general — el servidor "de toda la vida", administrado a mano (SO, parches, escalado). |
| **API Gateway** | Networking / API | Punto de entrada administrado para exponer APIs (REST/WebSocket) hacia Lambda, EC2 u otros backends — ver [API Gateway](../../backend/api-gateway.md) para el patrón genérico. |
| **Lambda** | Cómputo serverless | Ejecutar código sin gestionar servidores, factura por invocación — el FaaS de AWS, ver [El espectro IaaS → PaaS → Serverless](../../devops/vps-vs-cloud-run.md#el-espectro-completo-iaas--paas--serverless). |
| **S3** (Simple Storage Service) | Almacenamiento | Objetos (archivos, backups, hosting estático) — durabilidad muy alta, no pensado para queries relacionales. |
| **DynamoDB** | Base de datos NoSQL | Key-Value/documento gestionado, escala horizontal automática — ver [NoSQL](../../database/nosql.md). |
| **RDS** (Relational Database Service) | Base de datos SQL | Relacional administrada (Postgres, MySQL, etc.) — AWS se encarga de backups, patching, failover. |
| **CloudWatch** | Observabilidad | Logs, métricas y alarmas de todo lo que corre en la cuenta — el punto central de monitoreo. |
| **Cognito** | Identidad | Autenticación/autorización de usuarios gestionada (user pools, login social) — ver [Proveedores de Identidad Gestionados](../../backend/proveedores-de-identidad-gestionados.md). |
| **SNS** (Simple Notification Service) | Mensajería pub/sub | Fan-out: un mensaje, muchos suscriptores (email, SMS, colas, Lambda). |
| **SQS** (Simple Queue Service) | Mensajería (cola) | Desacoplar productor de consumidor con una cola — ver [Colas de mensajes](../../backend/colas-de-mensajes.md#sqs-amazon-simple-queue-service). |
| **ECS / EKS** | Orquestación de contenedores | ECS = orquestador propio de AWS; EKS = Kubernetes gestionado — ver [Kubernetes](../../devops/kubernetes.md). |
| **IAM** (Identity and Access Management) | Seguridad / Identidad | Quién (usuario, rol, servicio) puede hacer qué sobre qué recurso — la base de permisos de toda la cuenta. |

## Complementos (nice to have)

Conceptos que se piden con menos frecuencia que la tabla de arriba, pero vale la pena tener agendados — algunos son herramientas cloud-agnostic (no atadas a AWS), otros son networking de base.

| Concepto | Categoría | Qué es |
|---|---|---|
| **Kubernetes** | Orquestación de contenedores | Orquesta contenedores a escala (deploy, scaling, self-healing) — alcanza con nivel usuario/teórico, no operarlo a fondo. Ya desarrollado en [Kubernetes](../../devops/kubernetes.md) (readiness probes, HPA, resource limits). |
| **Terraform** | Infrastructure as Code (IaC) | Declarar infraestructura (servidores, redes, DBs) en archivos de config versionables, en vez de clickear en la consola a mano. |
| **AWS CDK** | Infrastructure as Code (IaC) | Lo mismo que Terraform, pero en un lenguaje de programación real (Python/TypeScript) en vez de HCL — compila a CloudFormation por detrás. |
| **Serverless Framework** | Deploy / IaC para serverless | Framework para definir y desplegar funciones Lambda + sus triggers (API Gateway, S3, etc.) con un solo archivo de config. |
| **Nginx** | Reverse proxy / web server | Termina TLS, sirve estáticos, hace de reverse proxy delante de la app. Ya desarrollado en [Nginx como reverse proxy](../../devops/deploy-vps.md#4-nginx-como-reverse-proxy). |
| **CDN (CloudFront)** | Networking / distribución de contenido | Cachea contenido cerca del usuario final — CloudFront es el CDN de AWS, ver [Comparación de Proveedores](../comparacion-proveedores.md) para el equivalente en Azure/GCP y [CDN](../../devops/cdn.md) para el concepto genérico. |
| **VPC** | Networking | Red virtual aislada donde corren los recursos de una cuenta cloud — ver [Comparación de Proveedores](../comparacion-proveedores.md). |
| **DNS** | Networking | Traduce nombres de dominio a IPs — ya desarrollado en [Resolución DNS](../../system-design/que-pasa-cuando-escribis-una-url.md#2-resolución-dns--de-dominio-a-ip). |

---
Relacionado: [Comparación de Proveedores](../comparacion-proveedores.md), [Backend](../../backend/), [Database](../../database/), [DevOps](../../devops/).
