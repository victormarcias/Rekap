# Dockerization (generic concepts)

How to package an app into a reproducible image, no matter the stack (Python, Node, Go...) or where it gets deployed afterward (VPS, Cloud Run, K8s). The concepts belong to Docker, not the language — the exact Dockerfile syntax is the only thing that changes from one stack to another.

## Virtualization — the origin of containers

Before containers, virtualizing meant running a full **virtual machine (VM)**: a *hypervisor* emulates hardware, and each VM runs its own entire operating system on top (kernel included). It isolates completely between VMs, but it's heavy — minutes to boot, GBs of RAM/disk per instance, just to have each one's OS running.

A container is much lighter because it does **not** virtualize hardware or run its own kernel: it shares the host operating system's kernel, and uses that kernel's native mechanisms (on Linux, *namespaces* to isolate what each process sees — its own filesystem, network, process list — and *cgroups* to limit how much CPU/RAM it can use) so each container feels isolated without needing its own full OS. That's why it starts in seconds/milliseconds instead of minutes, and weighs MBs instead of GBs.

```
VM                                Container
┌───────┬───────┐                 ┌───────┬───────┐
│ App A │ App B │                 │ App A │ App B │
├───────┼───────┤                 ├───────┴───────┤
│Guest OS│Guest OS│                │ Docker Engine  │
├───────┴───────┤                 ├───────────────┤
│  Hypervisor    │                 │ Host kernel    │  ← shared
├───────────────┤                 ├───────────────┤
│    Host OS     │                 │    Host OS     │
└───────────────┘                 └───────────────┘
```

**The real difference** isn't just "containers are lighter" — that's the consequence. The cause is the **shared kernel**: a VM isolates with emulated hardware and a full OS per instance; a container isolates with namespaces/cgroups on top of the same kernel, without duplicating the operating system.

## 1. Why containerize

Without Docker, "works on my machine" depends on which runtime version you have installed, which OS packages are present, and which environment variables you set by hand 3 months ago and don't remember anymore. A container packages the runtime, the dependencies, and the code into a single immutable image — what runs on your laptop is *exactly* what runs in production, byte for byte.

## 2. Multi-stage build

A normal build installs compilers and build tools (dev headers, package managers) that the app needs to *install* but not to *run* — and those tools end up shipping in the final image, bloating it and adding attack surface. A multi-stage build uses one image to compile/install, and copies only the final result into a clean image — the build tools never reach production.

*E.g. with Python (`uv`):*

```dockerfile
# ---- Stage 1: build ----
FROM python:3.12-slim AS builder
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen --no-dev

# ---- Stage 2: runtime ----
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv   # ✅ only the result, not the tools that generated it
COPY . .
ENV PATH="/app/.venv/bin:$PATH"
CMD ["fastapi", "run", "main.py", "--port", "8000"]
```

The final image has no `uv` and no install cache — just the finished venv and the code. The same pattern applies to any stack: in Node, the build stage runs `npm ci` and `npm run build`, and the final stage only copies `node_modules` (or the bundle) without the TypeScript compiler or the npm cache; in Go, the build stage compiles to a static binary, and the final stage doesn't even need the language runtime — it just copies that binary.

## 3. `.dockerignore`

Without this, `COPY . .` copies the **entire** directory into the image — including `.git/`, the locally installed virtual environment/dependencies, build caches, and any `.env` with secrets. Besides bloating the image, it's an easy way to leak credentials by accident.

```
# .dockerignore — adapt the "local deps/cache" list to the stack (.venv in Python, node_modules in Node, target/ in Rust)
.git
.venv
__pycache__
*.pyc
.env
.pytest_cache
```

## 4. Base image: `slim`/`alpine` vs full

"Full" base images (e.g. `python:3.12`, `node:20`) ship compilers and dev libraries that are almost never needed at runtime. The trimmed-down variants (`python:3.12-slim`, `node:20-alpine`) are a fraction of the size, with just the essentials to run the runtime — the trade-off is that if a dependency needs to compile something native (e.g. a library with a C extension), it can fail from a missing system header, and then you have to install it explicitly in the build stage.

*E.g. with Python:*

```dockerfile
# if a dependency needs to compile something native, add the minimum required in the builder stage:
RUN apt-get update && apt-get install -y --no-install-recommends gcc libpq-dev && rm -rf /var/lib/apt/lists/*
```

## 5. Local build and run

```bash
docker build -t myapp .
docker run -p 8000:8000 --env-file .env myapp
```

`--env-file .env` injects environment variables without hardcoding them into the image — how the app reads them depends on the stack (`pydantic-settings` in Python, `dotenv` in Node, system environment variables in Go), but Docker's mechanism is the same: the image itself has no embedded secrets, so it can be shared/pushed to a registry without leaking anything.

---
Related: [Deploy to Cloud Run](deploy-cloud-run.md), [Deploy to a VPS](deploy-vps.md), [FastAPI Authentication](../stacks/fastapi/autenticacion.md) (`pydantic-settings`, the example from point 5 in Python).
