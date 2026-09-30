# Azure — Servicios Principales

Mapa rápido de los servicios más nombrados de Azure y para qué sirve cada uno — varios ya tienen su propio desarrollo en otras carpetas del repo (se linkean en vez de repetir).

| Servicio | Categoría | Caso de uso típico |
|---|---|---|
| **Virtual Machines** | Cómputo | Máquina virtual de propósito general — el servidor "de toda la vida", administrado a mano (SO, parches, escalado). Equivalente al EC2 de AWS / Compute Engine de GCP. |
| **Azure Functions** | Cómputo serverless (FaaS) | Ejecutar una función sola sin gestionar servidores, factura por invocación — el equivalente a Lambda de Azure. |
| **App Service** | Hosting PaaS | Hosting gestionado para una app web/API completa (se encarga del SO, runtime, escalado) sin gestionar contenedores ni VMs vos mismo. |
| **Blob Storage** | Almacenamiento | Objetos (archivos, backups, hosting estático) — equivalente al S3 de AWS / Cloud Storage de GCP, con redundancia configurable (un solo datacenter, entre zonas, o geo-redundante). |
| **Cosmos DB** | Base de datos NoSQL (multi-modelo) | Base de datos gestionada con soporte para APIs de documento/key-value/grafo/columna con distribución global — ver [NoSQL](../../database/nosql.es.md). |
| **Azure SQL Database** | Base de datos SQL | Relacional administrada (motor SQL Server) — Azure se encarga de backups, patching, failover. |
| **Azure Monitor** | Observabilidad | Logs, métricas y alertas (Log Analytics y Application Insights viven bajo este paraguas) de todo lo que corre en la suscripción. |
| **Microsoft Entra ID** (antes Azure AD) | Identidad | Servicio de directorio detrás del login, registro de apps, y asignación de roles — ver [Proveedores de Identidad Gestionados](../../backend/managed-identity-providers.es.md). |
| **Service Bus** | Mensajería (cola + pub/sub) | Broker de mensajes que soporta tanto colas como topics/subscriptions (pub/sub) en un solo servicio. |
| **Event Grid** | Mensajería (eventing) | Ruteo de eventos liviano y reactivo — cambia un recurso, se avisa a los suscriptores. |
| **AKS** (Azure Kubernetes Service) | Orquestación de contenedores | Kubernetes gestionado — ver [Kubernetes](../../devops/kubernetes.es.md). |
| **RBAC** (Role-Based Access Control) | Seguridad / Identidad | Quién (usuario, grupo, service principal) puede hacer qué, con alcance a nivel management group / suscripción / resource group / recurso. |

## Complementos (nice to have)

Conceptos que se piden con menos frecuencia que la tabla de arriba, pero vale la pena tener agendados — algunos son herramientas cloud-agnostic (no atadas a Azure), otros son networking de base.

| Concepto | Categoría | Qué es |
|---|---|---|
| **Kubernetes** | Orquestación de contenedores | Orquesta contenedores a escala (deploy, scaling, self-healing) — alcanza con nivel usuario/teórico, no operarlo a fondo. Ya desarrollado en [Kubernetes](../../devops/kubernetes.es.md) (readiness probes, HPA, resource limits). |
| **ARM Templates / Bicep** | Infrastructure as Code (IaC) | El IaC nativo de Azure — Bicep es un DSL más limpio que compila a templates ARM (JSON). Terraform también se usa mucho en Azure. |
| **Azure CDN** | Networking / distribución de contenido | Cachea contenido cerca del usuario final — el CDN de Azure, ver [Comparación de Proveedores](../provider-comparison.es.md) para el equivalente en AWS/GCP y [CDN](../../devops/cdn.es.md) para el concepto genérico. |
| **Virtual Network (VNet)** | Networking | Red virtual aislada donde corren los recursos de una suscripción — ver [Comparación de Proveedores](../provider-comparison.es.md). |
| **Azure DNS** | Networking | Traduce nombres de dominio a IPs — ya desarrollado en [Resolución DNS](../../system-design/what-happens-when-you-type-a-url.es.md#2-resolución-dns--de-dominio-a-ip). |

---
Relacionado: [Comparación de Proveedores](../provider-comparison.es.md), [Backend](../../backend/README.es.md), [Database](../../database/README.es.md), [DevOps](../../devops/).
