# GCP — Servicios Principales

Mapa rápido de los servicios más nombrados de GCP y para qué sirve cada uno — varios ya tienen su propio desarrollo en otras carpetas del repo (se linkean en vez de repetir).

| Servicio | Categoría | Caso de uso típico |
|---|---|---|
| **Compute Engine** | Cómputo | Máquina virtual de propósito general — el servidor "de toda la vida", administrado a mano (SO, parches, escalado). Equivalente al EC2 de AWS. |
| **Cloud Run** | Contenedores serverless | Corre un contenedor sin gestionar servidores, escala a cero — ver [Deploy a Cloud Run](../../devops/deploy-cloud-run.es.md) y [VPS vs Cloud Run](../../devops/vps-vs-cloud-run.es.md). |
| **Cloud Functions** | Cómputo serverless (FaaS) | Ejecutar una función sola sin gestionar servidores, factura por invocación — el equivalente a Lambda de GCP, ver [El espectro IaaS → PaaS → Serverless](../../devops/vps-vs-cloud-run.es.md#el-espectro-completo-iaas--paas--serverless). |
| **Cloud Storage** | Almacenamiento | Objetos (archivos, backups, hosting estático) — equivalente al S3 de AWS, durabilidad muy alta, no pensado para queries relacionales. |
| **Firestore / Bigtable** | Base de datos NoSQL | Firestore: documento gestionado para datos de app. Bigtable: columnar ancho para workloads analíticos/time-series a gran escala — ver [NoSQL](../../database/nosql.es.md). |
| **Cloud SQL** | Base de datos SQL | Relacional administrada (Postgres, MySQL, SQL Server) — GCP se encarga de backups, patching, failover. |
| **BigQuery** | Data warehouse | Analytics SQL serverless sobre datasets grandes — sin servidores para aprovisionar, factura por query o por capacidad reservada. |
| **Pub/Sub** | Mensajería (pub/sub) | Mensajería fan-out con entrega durable — los publishers mandan mensajes, los suscriptores hacen pull a su propio ritmo. |
| **Dataflow** | Procesamiento stream/batch | Apache Beam gestionado — el mismo código de pipeline corre para procesamiento tanto batch como streaming. |
| **Dataproc** | Hadoop/Spark gestionado | Clusters de Hadoop/Spark gestionados, típicamente usados por equipos migrando un cluster on-prem existente al cloud. |
| **GKE** (Google Kubernetes Engine) | Orquestación de contenedores | Kubernetes gestionado — ver [Kubernetes](../../devops/kubernetes.es.md). |
| **IAM** (Identity and Access Management) | Seguridad / Identidad | Quién (usuario, service account, grupo) puede hacer qué sobre qué recurso, vía roles básicos, predefinidos o custom. |

## Complementos (nice to have)

Conceptos que se piden con menos frecuencia que la tabla de arriba, pero vale la pena tener agendados — algunos son herramientas cloud-agnostic (no atadas a GCP), otros son networking de base.

| Concepto | Categoría | Qué es |
|---|---|---|
| **Kubernetes** | Orquestación de contenedores | Orquesta contenedores a escala (deploy, scaling, self-healing) — alcanza con nivel usuario/teórico, no operarlo a fondo. Ya desarrollado en [Kubernetes](../../devops/kubernetes.es.md) (readiness probes, HPA, resource limits). |
| **Terraform** | Infrastructure as Code (IaC) | Declarar infraestructura (VMs, redes, DBs) en archivos de config versionables, en vez de clickear en la consola a mano — la herramienta de IaC dominante en GCP. |
| **Cloud CDN** | Networking / distribución de contenido | Cachea contenido cerca del usuario final — el CDN de GCP, ver [Comparación de Proveedores](../provider-comparison.es.md) para el equivalente en AWS/Azure y [CDN](../../devops/cdn.es.md) para el concepto genérico. |
| **VPC** | Networking | Red virtual aislada donde corren los recursos de un proyecto cloud — ver [Comparación de Proveedores](../provider-comparison.es.md). |
| **Cloud DNS** | Networking | Traduce nombres de dominio a IPs — ya desarrollado en [Resolución DNS](../../system-design/what-happens-when-you-type-a-url.es.md#2-resolución-dns--de-dominio-a-ip). |

---
Relacionado: [Comparación de Proveedores](../provider-comparison.es.md), [Backend](../../backend/README.es.md), [Database](../../database/README.es.md), [DevOps](../../devops/).
