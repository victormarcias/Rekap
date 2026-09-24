# Security Headers

El servidor no agrega headers de seguridad por sí solo — hay que declararlos explícitamente en cada respuesta (ver el [middleware de ejemplo en Deploy a Cloud Run](../devops/deploy-cloud-run.es.md#5-security-headers-vía-middleware) para la implementación concreta). Acá el detalle de qué previene cada uno.

## Content-Security-Policy (CSP)

Restringe de qué orígenes puede la página cargar y ejecutar scripts, estilos, imágenes, etc. Es la defensa que mitiga el impacto de un [XSS](xss.es.md) aunque el escape falle en algún lugar puntual — incluso si un script malicioso logra inyectarse, el navegador no lo ejecuta si viene de un origen no permitido.

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.confiable.com
```

## X-Frame-Options / `frame-ancestors` (CSP)

Evita que el sitio se cargue dentro de un `<iframe>` de otro dominio — mitiga **clickjacking**: el atacante superpone tu sitio (invisible, con opacidad 0) sobre una página propia, para que el usuario crea que hace click en algo inofensivo cuando en realidad hace click en un botón tuyo (ej. "Confirmar transferencia").

```
X-Frame-Options: DENY
```

## X-Content-Type-Options: nosniff

Evita que el navegador intente "adivinar" el tipo de contenido de una respuesta e ignore el `Content-Type` declarado — sin esto, un archivo subido como imagen pero que en realidad contiene JavaScript podría terminar ejecutándose como script en vez de mostrarse como imagen.

```
X-Content-Type-Options: nosniff
```

## Strict-Transport-Security (HSTS)

Le dice al navegador que fuerce HTTPS en **todos** los requests futuros a ese dominio, incluso si el usuario escribe `http://` a mano o hace click en un link viejo con `http`. Sin esto, cada visita queda expuesta a un downgrade a HTTP en la primera conexión (y a un ataque man-in-the-middle en esa ventana).

```
Strict-Transport-Security: max-age=63072000; includeSubDomains
```

---
Relacionado: [XSS](xss.es.md), [Deploy a Cloud Run](../devops/deploy-cloud-run.es.md#5-security-headers-vía-middleware).
