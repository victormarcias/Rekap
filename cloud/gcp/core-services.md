# GCP — Core Services

Quick map of GCP's most commonly referenced services and what each one is for — several already have their own writeup elsewhere in the repo (linked instead of repeated).

| Service | Category | Typical use case |
|---|---|---|
| **Compute Engine** | Compute | General-purpose virtual machine — the "old school" server, managed by hand (OS, patches, scaling). AWS's EC2 equivalent. |
| **Cloud Run** | Serverless containers | Runs a container without managing servers, scales to zero — see [Deploy to Cloud Run](../../devops/deploy-cloud-run.md) and [VPS vs Cloud Run](../../devops/vps-vs-cloud-run.md). |
| **Cloud Functions** | Serverless compute (FaaS) | Run a single function without managing servers, billed per invocation — GCP's Lambda equivalent, see [The IaaS → PaaS → Serverless spectrum](../../devops/vps-vs-cloud-run.md#the-full-spectrum-iaas--paas--serverless). |
| **Cloud Storage** | Storage | Objects (files, backups, static hosting) — AWS S3 equivalent, very high durability, not built for relational queries. |
| **Firestore / Bigtable** | NoSQL database | Firestore: managed document store for app data. Bigtable: wide-column store for large-scale analytical/time-series workloads — see [NoSQL](../../database/nosql.md). |
| **Cloud SQL** | SQL database | Managed relational database (Postgres, MySQL, SQL Server) — GCP handles backups, patching, failover. |
| **BigQuery** | Data warehouse | Serverless SQL analytics over large datasets — no servers to provision, billed per query or per reserved capacity. |
| **Pub/Sub** | Messaging (pub/sub) | Fan-out messaging with durable delivery — publishers send messages, subscribers pull them at their own pace. |
| **Dataflow** | Stream/batch processing | Managed Apache Beam — the same pipeline code runs for both batch and streaming data processing. |
| **Dataproc** | Managed Hadoop/Spark | Managed Hadoop/Spark clusters, typically used by teams migrating an existing on-prem cluster to the cloud. |
| **Cloud Load Balancing** | Networking / load balancing | Managed [load balancer](../../backend/load-balancers.md#managed-load-balancers-cloud), L4 (Network) and L7 (Application), can be global: one IP in front of backends in several regions. |
| **GKE** (Google Kubernetes Engine) | Container orchestration | Managed Kubernetes — see [Kubernetes](../../devops/kubernetes.md). |
| **IAM** (Identity and Access Management) | Security / Identity | Who (user, service account, group) can do what on which resource, via basic, predefined, or custom roles. |

## Nice-to-have extras

Concepts asked about less often than the table above, but worth keeping in mind — some are cloud-agnostic tools (not tied to GCP), others are baseline networking.

| Concept | Category | What it is |
|---|---|---|
| **Kubernetes** | Container orchestration | Orchestrates containers at scale (deploy, scaling, self-healing) — user/theory level is enough, no need to operate it in depth. Already covered in [Kubernetes](../../devops/kubernetes.md) (readiness probes, HPA, resource limits). |
| **Terraform** | Infrastructure as Code (IaC) | Declares infrastructure (VMs, networks, DBs) in versionable config files, instead of clicking through the console by hand — the dominant IaC tool on GCP. |
| **Cloud CDN** | Networking / content delivery | Caches content close to the end user — GCP's CDN, see [Provider Comparison](../provider-comparison.md) for the AWS/Azure equivalent and [CDN](../../devops/cdn.md) for the generic concept. |
| **VPC** | Networking | Isolated virtual network where a cloud project's resources run — see [Provider Comparison](../provider-comparison.md). |
| **Cloud DNS** | Networking | Translates domain names to IPs — already covered in [DNS Resolution](../../system-design/what-happens-when-you-type-a-url.md#2-dns-resolution--from-domain-to-ip). |

---
Related: [Provider Comparison](../provider-comparison.md), [Backend](../../backend/), [Database](../../database/), [DevOps](../../devops/).
