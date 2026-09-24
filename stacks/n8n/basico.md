# n8n — Fundamentos

## Qué es

Herramienta de automatización de workflows, **node-based** (arrastrar y conectar bloques visuales) y **open source** — alternativa a Zapier/Make pensada para gente técnica: se puede self-hostear gratis, y trae un nodo de código (JS/Python) para lógica custom que las alternativas puramente no-code no ofrecen. Un workflow conecta un **trigger** (qué dispara la ejecución) con una cadena de **acciones** (llamar una API, mandar un email, escribir en una DB, transformar datos).

## Dónde corre: Local vs Self-hosted (VPS) vs Cloud

- **Local**: correr n8n en tu máquina (Docker o `npm install n8n`) para desarrollar y probar — no accesible desde afuera de tu red sin exponerlo.
- **Self-hosted (VPS)**: tu propia instancia en un servidor propio (ver [Deploy a un VPS](../../devops/deploy-vps.es.md)) — control total, sin los límites de un plan gratuito, pero administrás vos el uptime, los backups y las actualizaciones.
- **n8n Cloud**: versión gestionada por los creadores de n8n — sin infraestructura propia que mantener, a cambio de un plan pago según uso.

La elección es el mismo trade-off que ya vimos entre [VPS y Cloud Run](../../devops/vps-vs-cloud-run.es.md): cuánto control querés vs cuánto mantenimiento estás dispuesto a asumir.

## Cómo funciona: nodes, triggers y JSON

Cada **nodo** es un paso del workflow — una acción, una condición, una transformación de datos. El **trigger node** es el que dispara todo (un webhook entrante, un cron cada X minutos, un evento de una app conectada). Los nodos se conectan en cadena, y lo que pasa entre ellos es siempre **JSON**: el output de un nodo es el input del siguiente.

```json
// output típico de un nodo (ej. después de llamar una API) —
// esto es exactamente lo que recibe el nodo siguiente como input
{
  "id": 123,
  "name": "Ana",
  "email": "ana@mail.com"
}
```

n8n trae cientos de integraciones prearmadas (Slack, Gmail, Google Sheets, bases de datos, etc.), pero el nodo más versátil es **HTTP Request** — llama a cualquier API REST que no tenga una integración nativa, con los mismos verbos/headers/body que armarías a mano (ver [REST](../../backend/rest.es.md), [HTTP Methods](../../backend/http-methods.es.md)).

## Configuración: tres cosas distintas que se llaman "config"

**1. El workflow en sí — JSON.** Todos sus nodos, conexiones y parámetros se guardan y exportan como un único archivo JSON — es lo que versionás en git, compartís con el equipo, o migrás entre instancias. No confundir con el JSON que pasa *entre* nodos en runtime (arriba) — esto es la definición del workflow completo.

```json
{
  "name": "Procesar pedido nuevo",
  "nodes": [
    { "id": "1", "type": "n8n-nodes-base.webhook", "parameters": { "path": "nuevo-pedido" } },
    { "id": "2", "type": "n8n-nodes-base.httpRequest", "parameters": { "url": "https://api.miapp.com/pedidos" } }
  ],
  "connections": {
    "1": { "main": [[{ "node": "2", "type": "main", "index": 0 }]] }
  }
}
```

**2. La instancia del servidor — variables de entorno (`.env`).** Cómo se configura el servidor de n8n en sí (no un workflow puntual) no usa un archivo de config propio — son variables de entorno estándar.

```bash
# .env
N8N_PORT=5678
N8N_HOST=n8n.miapp.com
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
N8N_ENCRYPTION_KEY=una-clave-secreta
```

**3. Si se self-hostea con Docker — `docker-compose.yml`.** El YAML no es de n8n, es de Docker Compose — orquesta el contenedor, sus variables de entorno y los volúmenes donde persiste la data.

```yaml
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_PORT=5678
      - DB_TYPE=postgresdb
    volumes:
      - ~/.n8n:/home/node/.n8n
```

Un `package.json` solo aparece si se instala n8n vía `npm install n8n` en vez de Docker — ahí es gestión de paquetes estándar de Node, sin nada específico de n8n.

---
Relacionado: [n8n y Agentic AI](n8n-y-agentic.md), [Cómo se testea](testing.md), [Deploy a un VPS](../../devops/deploy-vps.es.md), [VPS vs Cloud Run](../../devops/vps-vs-cloud-run.es.md), [REST](../../backend/rest.es.md).
