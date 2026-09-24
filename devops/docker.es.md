# Dockerización (conceptos genéricos)

Cómo empaquetar una app en una imagen reproducible, sin importar de qué stack se trate (Python, Node, Go...) ni a dónde se despliegue después (VPS, Cloud Run, K8s). Los conceptos son de Docker, no del lenguaje — la sintaxis exacta del Dockerfile es lo único que cambia de un stack a otro.

## Virtualización — el origen de los contenedores

Antes de los contenedores, virtualizar significaba correr una **máquina virtual (VM)** completa: un *hypervisor* emula hardware, y cada VM corre arriba su propio sistema operativo entero (kernel incluido). Aísla por completo entre VMs, pero es pesado — minutos para arrancar, GBs de RAM/disco por instancia, solo para tener el SO de cada una corriendo.

Un contenedor es mucho más liviano porque **no** virtualiza hardware ni corre un kernel propio: comparte el kernel del sistema operativo host, y usa mecanismos nativos de ese kernel (en Linux, *namespaces* para aislar qué ve cada proceso — su propio filesystem, red, lista de procesos — y *cgroups* para limitar cuánto CPU/RAM puede usar) para que cada contenedor se sienta aislado sin necesitar su propio SO completo. Por eso arranca en segundos/milisegundos en vez de minutos, y pesa MBs en vez de GBs.

```
VM                                Container
┌───────┬───────┐                 ┌───────┬───────┐
│ App A │ App B │                 │ App A │ App B │
├───────┼───────┤                 ├───────┴───────┤
│Guest OS│Guest OS│                │ Docker Engine  │
├───────┴───────┤                 ├───────────────┤
│  Hypervisor    │                 │ Kernel del host│  ← compartido
├───────────────┤                 ├───────────────┤
│    Host OS     │                 │    Host OS     │
└───────────────┘                 └───────────────┘
```

**La diferencia real** no es solo "los containers son más livianos" — eso es la consecuencia. La causa es el **kernel compartido**: una VM aísla con hardware emulado y un SO completo por instancia; un container aísla con namespaces/cgroups sobre el mismo kernel, sin duplicar el sistema operativo.

## 1. Por qué dockerizar

Sin Docker, "funciona en mi máquina" depende de qué versión del runtime tenés instalada, qué paquetes del sistema operativo están presentes, y qué variables de entorno seteaste a mano hace 3 meses y ya no te acordás. Un contenedor empaqueta el runtime, las dependencias y el código en una sola imagen inmutable — lo que corre en tu laptop es *exactamente* lo que corre en producción, byte por byte.

## 2. Multi-stage build

Un build normal instala compiladores y herramientas de build (headers de desarrollo, gestores de paquetes) que la app necesita para *instalarse* pero no para *correr* — y esas herramientas terminan viajando a la imagen final, infladas y con más superficie de ataque. Un multi-stage build usa una imagen para compilar/instalar, y copia solo el resultado final a una imagen limpia — las herramientas de build nunca llegan a producción.

*Ej. con Python (`uv`):*

```dockerfile
# ---- Stage 1: build ----
FROM python:3.12-slim AS builder
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen --no-dev

# ---- Stage 2: runtime ----
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv   # ✅ solo el resultado, no las herramientas que lo generaron
COPY . .
ENV PATH="/app/.venv/bin:$PATH"
CMD ["fastapi", "run", "main.py", "--port", "8000"]
```

La imagen final no tiene `uv` ni ningún cache de instalación — solo el venv ya armado y el código. El mismo patrón aplica en cualquier stack: en Node, el stage de build corre `npm ci` y `npm run build`, y el stage final solo copia `node_modules` (o el bundle) sin el compilador de TypeScript ni el cache de npm; en Go, el stage de build compila a un binario estático, y el stage final ni siquiera necesita el runtime del lenguaje — solo copia ese binario.

## 3. `.dockerignore`

Sin esto, `COPY . .` copia **todo** el directorio a la imagen — incluyendo `.git/`, el entorno virtual/dependencias instaladas localmente, cachés de build, y cualquier `.env` con secretos. Además de inflar la imagen, es una forma fácil de filtrar credenciales sin querer.

```
# .dockerignore — adaptar la lista de "dependencias/cache local" al stack (.venv en Python, node_modules en Node, target/ en Rust)
.git
.venv
__pycache__
*.pyc
.env
.pytest_cache
```

## 4. Imagen base: `slim`/`alpine` vs completa

Las imágenes base "completas" (ej. `python:3.12`, `node:20`) traen compiladores y librerías de desarrollo que casi nunca hacen falta en runtime. Las variantes reducidas (`python:3.12-slim`, `node:20-alpine`) son una fracción del tamaño, con solo lo esencial para correr el runtime — el trade-off es que si una dependencia necesita compilar algo nativo (ej. una librería con extensión en C), puede fallar por faltarle un header del sistema, y ahí hay que instalarlo explícitamente en el stage de build.

*Ej. con Python:*

```dockerfile
# si una dependencia necesita compilar algo nativo, agregar lo mínimo necesario en el builder stage:
RUN apt-get update && apt-get install -y --no-install-recommends gcc libpq-dev && rm -rf /var/lib/apt/lists/*
```

## 5. Build y run local

```bash
docker build -t miapp .
docker run -p 8000:8000 --env-file .env miapp
```

`--env-file .env` inyecta variables de entorno sin hardcodearlas en la imagen — cómo las lee la app depende del stack (`pydantic-settings` en Python, `dotenv` en Node, variables de entorno del sistema en Go) pero el mecanismo de Docker es el mismo: la imagen en sí no tiene ningún secreto embebido, así que se puede compartir/pushear a un registry sin filtrar nada.

---
Relacionado: [Deploy a Cloud Run](deploy-cloud-run.es.md), [Deploy a un VPS](deploy-vps.es.md), [Autenticación en FastAPI](../stacks/fastapi/autenticacion.md) (`pydantic-settings`, ejemplo del punto 5 en Python).
