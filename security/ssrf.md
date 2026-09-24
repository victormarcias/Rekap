# SSRF (Server-Side Request Forgery)

Engañar al **servidor** para que haga una request HTTP a una URL que el atacante elige — típico cuando la app acepta una URL provista por el usuario y el backend la "visita" (descargar un avatar desde una URL, generar una miniatura, validar la URL de un webhook).

```python
# ❌ vulnerable: el servidor busca cualquier URL que le pase el usuario, sin restricción
@app.post("/avatar-desde-url")
def avatar_desde_url(url: str):
    imagen = requests.get(url)
    return guardar_imagen(imagen.content)

# el atacante manda url="http://169.254.169.254/latest/meta-data/iam/security-credentials/"
# (el endpoint interno de metadata de AWS) — el servidor SÍ tiene acceso de red ahí,
# y termina trayendo credenciales que nunca deberían salir de la infraestructura
```

## Por qué es especialmente grave en cloud

El servidor tiene acceso de red a recursos internos (endpoints de metadata del cloud provider, bases de datos internas, otros microservicios de la misma red privada) que un atacante, parado afuera, no podría alcanzar directamente. El SSRF usa al servidor como **proxy hacia adentro** de la red — el request "sale" del servidor, pero apunta para adentro.

## Cómo se previene

- **Whitelist de dominios/IPs permitidos**, en vez de aceptar cualquier URL que mande el usuario.
- **Bloquear rangos de IP privados/internos** (`169.254.0.0/16`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.1`) en cualquier request que el servidor arme a partir de un input externo.
- **Deshabilitar redirects automáticos** al hacer el fetch — una URL "válida" a simple vista puede redirigir a una interna, y si el cliente HTTP sigue redirects ciegamente, la validación de la URL original no sirve de nada.

---
Relacionado: [SQL Injection](sql-injection.md), [AWS — Servicios Principales](../cloud/aws/core-services.es.md) (metadata endpoint), [Dockerización](../devops/docker.md).
