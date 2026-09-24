# VPS vs Cloud Run

## What a VPS is

**Virtual Private Server**: a slice of a large physical server, virtualized to behave like its own independent machine — root access, your own OS, guaranteed resources (CPU/RAM/disk) isolated from the rest, even though the underlying physical hardware is shared with other VPS's on the same server (see [Virtualization](docker.md#virtualization--the-origin-of-containers) for the technical mechanism). It's different from:

- **Shared hosting**: there you don't even have your own OS — you share the same process/environment with other customers, with no root access.
- **Dedicated server**: full physical hardware just for you, no virtualization — more expensive, without the limits that sharing hardware imposes.

Typical providers: DigitalOcean, Linode/Akamai, Hetzner, [AWS EC2](../cloud/aws/core-services.md).

Comparison between [Deploy to a VPS](deploy-vps.md) (your own server, always on) and [Deploy to Cloud Run](deploy-cloud-run.md) (serverless container, scales to zero).

## The full spectrum: IaaS → PaaS → Serverless

VPS and Cloud Run are two points on a wider spectrum — as you move along it, you delegate more responsibility to the platform, in exchange for less control:

| Model | What it is | Examples | Typical use case |
|---|---|---|---|
| **IaaS** (Infrastructure as a Service) | An empty virtual machine — you install the OS, the runtime, everything. Maximum control, maximum responsibility. | [AWS EC2](../cloud/aws/core-services.md), a VPS | You need total control, or something that doesn't fit into a container/function. |
| **PaaS** (Platform as a Service) | You hand over your code (or a `git push`), the platform takes care of the OS, the runtime, and the deploy — you no longer touch a server directly. | Heroku, Render, Railway | You don't want to deal with servers without paying the cost of cold starts — it runs always (like a VPS), but tends to cost more than an equivalent VPS at high, constant traffic. |
| **Serverless / CaaS** (Container as a Service) | You hand over a container, the platform decides how many instances to run and when, including scaling to zero. | Google Cloud Run | Variable/intermittent traffic — you accept cold starts in exchange for paying $0 when there's no traffic. |
| **FaaS** (Function as a Service) | Not even a container — a single function that the platform runs on demand. | [AWS Lambda](../cloud/aws/core-services.md) | One-off event-driven tasks (processing a file, answering a webhook), with nothing kept running. |
| **SaaS** (Software as a Service) | A different axis from the four above: using someone else's finished software, without deploying anything of your own. | Gmail, Salesforce | The need is "use a tool," not "build/deploy something of your own." |

## Technical comparison

| | VPS | Cloud Run |
|---|---|---|
| Who manages the OS | You (patches, firewall, SSH) | Google |
| Cost with zero traffic | Full price, 24/7 | ~$0 (scales to zero) |
| Cold start | Doesn't exist (always on) | Yes, on the first request after being at zero |
| HTTPS certificate | Manual (`certbot`, renewed every 90 days) | Automatic |
| Fine-grained infrastructure control | Total | Limited to what the platform exposes |

## Pros and cons

| | VPS | Cloud Run |
|---|---|---|
| **Pros** | Fixed, predictable cost at high, constant traffic; total control (install whatever, tune the kernel, run several services on the same machine); no cold start; not locked into a specific provider | Zero OS/patch maintenance; scales automatically with demand (and to zero when there's no traffic); HTTPS and load balancing handled by the platform; you pay for actual usage, not reserved capacity |
| **Cons** | You're responsible for security, updates, and capacity planning; you pay the same whether there's traffic or not; scaling means provisioning more machines by hand (or building your own autoscaling) | Cold start on sporadic traffic; less control (no access to the underlying OS); at high, sustained traffic it can end up costing more than your own machine; locked into the platform's conventions (request time limits, image size, etc.) |

---
Related: [Deploy to a VPS](deploy-vps.md), [Deploy to Cloud Run](deploy-cloud-run.md), [Cold start](../diagnostics/devops.md), [Elasticity](../system-design/quality-attributes.md#elasticity).
