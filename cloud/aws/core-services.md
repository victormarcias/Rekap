# AWS — Core Services

Quick map of AWS's most commonly referenced services and what each one is for — several already have their own writeup elsewhere in the repo (linked instead of repeated).

| Service | Category | Typical use case |
|---|---|---|
| **EC2** (Elastic Compute Cloud) | Compute | General-purpose virtual machine — the "old school" server, managed by hand (OS, patches, scaling). |
| **API Gateway** | Networking / API | Managed entry point for exposing APIs (REST/WebSocket) to Lambda, EC2, or other backends — see [API Gateway](../../backend/api-gateway.md) for the generic pattern. |
| **Lambda** | Serverless compute | Run code without managing servers, billed per invocation — AWS's FaaS, see [The IaaS → PaaS → Serverless spectrum](../../devops/vps-vs-cloud-run.md#the-full-spectrum-iaas--paas--serverless). |
| **S3** (Simple Storage Service) | Storage | Objects (files, backups, static hosting) — very high durability, not built for relational queries. |
| **DynamoDB** | NoSQL database | Managed Key-Value/document store, automatic horizontal scaling — see [NoSQL](../../database/nosql.md). |
| **RDS** (Relational Database Service) | SQL database | Managed relational database (Postgres, MySQL, etc.) — AWS handles backups, patching, failover. |
| **CloudWatch** | Observability | Logs, metrics, and alarms for everything running in the account — the central monitoring point. |
| **Cognito** | Identity | Managed user authentication/authorization (user pools, social login) — see [Managed Identity Providers](../../backend/managed-identity-providers.md). |
| **SNS** (Simple Notification Service) | Pub/sub messaging | Fan-out: one message, many subscribers (email, SMS, queues, Lambda). |
| **SQS** (Simple Queue Service) | Messaging (queue) | Decouples producer from consumer with a queue — see [Message Queues](../../backend/message-queues.md#sqs-amazon-simple-queue-service). |
| **ECS / EKS** | Container orchestration | ECS = AWS's own orchestrator; EKS = managed Kubernetes — see [Kubernetes](../../devops/kubernetes.md). |
| **IAM** (Identity and Access Management) | Security / Identity | Who (user, role, service) can do what on which resource — the permission base for the whole account. |

## Nice-to-have extras

Concepts asked about less often than the table above, but worth keeping in mind — some are cloud-agnostic tools (not tied to AWS), others are baseline networking.

| Concept | Category | What it is |
|---|---|---|
| **Kubernetes** | Container orchestration | Orchestrates containers at scale (deploy, scaling, self-healing) — user/theory level is enough, no need to operate it in depth. Already covered in [Kubernetes](../../devops/kubernetes.md) (readiness probes, HPA, resource limits). |
| **Terraform** | Infrastructure as Code (IaC) | Declares infrastructure (servers, networks, DBs) in versionable config files, instead of clicking through the console by hand. |
| **AWS CDK** | Infrastructure as Code (IaC) | Same idea as Terraform, but in a real programming language (Python/TypeScript) instead of HCL — compiles to CloudFormation under the hood. |
| **Serverless Framework** | Deploy / IaC for serverless | Framework for defining and deploying Lambda functions + their triggers (API Gateway, S3, etc.) with a single config file. |
| **Nginx** | Reverse proxy / web server | Terminates TLS, serves static files, acts as a reverse proxy in front of the app. Already covered in [Nginx as reverse proxy](../../devops/deploy-vps.md#4-nginx-as-reverse-proxy). |
| **CDN (CloudFront)** | Networking / content delivery | Caches content close to the end user — CloudFront is AWS's CDN, see [Provider Comparison](../provider-comparison.md) for the Azure/GCP equivalent and [CDN](../../devops/cdn.md) for the generic concept. |
| **VPC** | Networking | Isolated virtual network where a cloud account's resources run — see [Provider Comparison](../provider-comparison.md). |
| **DNS** | Networking | Translates domain names to IPs — already covered in [DNS Resolution](../../system-design/what-happens-when-you-type-a-url.md#2-dns-resolution--from-domain-to-ip). |

---
Related: [Provider Comparison](../provider-comparison.md), [Backend](../../backend/), [Database](../../database/), [DevOps](../../devops/).
