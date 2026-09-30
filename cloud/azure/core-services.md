# Azure — Core Services

Quick map of Azure's most commonly referenced services and what each one is for — several already have their own writeup elsewhere in the repo (linked instead of repeated).

| Service | Category | Typical use case |
|---|---|---|
| **Virtual Machines** | Compute | General-purpose virtual machine — the "old school" server, managed by hand (OS, patches, scaling). AWS EC2 / GCP Compute Engine equivalent. |
| **Azure Functions** | Serverless compute (FaaS) | Run a single function without managing servers, billed per invocation — Azure's Lambda equivalent. |
| **App Service** | PaaS hosting | Managed hosting for a full web app/API (handles the OS, runtime, scaling) without managing containers or VMs yourself. |
| **Blob Storage** | Storage | Objects (files, backups, static hosting) — AWS S3 / GCP Cloud Storage equivalent, with configurable redundancy (single datacenter, zone-redundant, or geo-redundant). |
| **Cosmos DB** | NoSQL database (multi-model) | Managed database supporting document/key-value/graph/column APIs with global distribution — see [NoSQL](../../database/nosql.md). |
| **Azure SQL Database** | SQL database | Managed relational database (SQL Server engine) — Azure handles backups, patching, failover. |
| **Azure Monitor** | Observability | Logs, metrics, and alerts (Log Analytics + Application Insights live under this umbrella) for everything running in the subscription. |
| **Microsoft Entra ID** (formerly Azure AD) | Identity | Directory service backing sign-in, app registrations, and role assignments — see [Managed Identity Providers](../../backend/managed-identity-providers.md). |
| **Service Bus** | Messaging (queue + pub/sub) | Message broker supporting both queues and topics/subscriptions (pub/sub) in one service. |
| **Event Grid** | Messaging (eventing) | Lightweight, reactive event routing — a resource changes, subscribers get notified. |
| **AKS** (Azure Kubernetes Service) | Container orchestration | Managed Kubernetes — see [Kubernetes](../../devops/kubernetes.md). |
| **RBAC** (Role-Based Access Control) | Security / Identity | Who (user, group, service principal) can do what, scoped at management group / subscription / resource group / resource level. |

## Nice-to-have extras

Concepts asked about less often than the table above, but worth keeping in mind — some are cloud-agnostic tools (not tied to Azure), others are baseline networking.

| Concept | Category | What it is |
|---|---|---|
| **Kubernetes** | Container orchestration | Orchestrates containers at scale (deploy, scaling, self-healing) — user/theory level is enough, no need to operate it in depth. Already covered in [Kubernetes](../../devops/kubernetes.md) (readiness probes, HPA, resource limits). |
| **ARM Templates / Bicep** | Infrastructure as Code (IaC) | Azure's native IaC — Bicep is a cleaner DSL that compiles down to ARM (JSON) templates. Terraform is also widely used on Azure. |
| **Azure CDN** | Networking / content delivery | Caches content close to the end user — Azure's CDN, see [Provider Comparison](../provider-comparison.md) for the AWS/GCP equivalent and [CDN](../../devops/cdn.md) for the generic concept. |
| **Virtual Network (VNet)** | Networking | Isolated virtual network where a subscription's resources run — see [Provider Comparison](../provider-comparison.md). |
| **Azure DNS** | Networking | Translates domain names to IPs — already covered in [DNS Resolution](../../system-design/what-happens-when-you-type-a-url.md#2-dns-resolution--from-domain-to-ip). |

---
Related: [Provider Comparison](../provider-comparison.md), [Backend](../../backend/), [Database](../../database/), [DevOps](../../devops/).
