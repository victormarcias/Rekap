# Privacidad y GDPR

**GDPR** (General Data Protection Regulation) es la regulación de protección de datos de la Unión Europea — pero en la práctica importa para casi cualquier app con usuarios internacionales: si un solo usuario en la UE usa tu producto, GDPR aplica, sin importar dónde esté la empresa. Se volvió el estándar de facto que otras leyes de privacidad (CCPA en California, LGPD en Brasil) copiaron en buena medida.

## Data Security

### Data Minimization

Recolectar solo el dato que **realmente se necesita**, no todo lo que "podría servir después". Un formulario de registro que pide teléfono, dirección y fecha de nacimiento cuando solo hace falta el email es más superficie de ataque en caso de breach, y más trabajo de compliance sin ningún beneficio a cambio.

```jsx
// ❌ pide más de lo que la feature actual necesita
<Form fields={['email', 'password', 'phone', 'address', 'birthdate']} />

// ✅ solo lo indispensable para el registro — el resto se pide después, si hace falta
<Form fields={['email', 'password']} />
```

### Data Encryption

Dato sensible cifrado tanto **en tránsito** (HTTPS/TLS — ver [TLS handshake](../system-design/what-happens-when-you-type-a-url.es.md#4-tls-handshake-si-es-https)) como **en reposo** (campos sensibles cifrados en la base de datos, no solo la conexión). Encriptar no es lo mismo que hashear — una contraseña se hashea (no se puede revertir), un dato que sí necesitás recuperar después (ej. un número de tarjeta) se encripta (ver [Hashing vs Encriptado vs Encoding](../backend/authentication.es.md#1-hashing-vs-encriptado-vs-encoding)).

### Correct Settings (configuración segura por default)

El error más común no es una vulnerabilidad de código, es una configuración por default insegura: un bucket de almacenamiento público cuando debía ser privado, una API key con permisos de administrador cuando solo necesitaba leer, headers de seguridad faltantes (ver [Security headers](../devops/deploy-cloud-run.es.md#5-security-headers-vía-middleware)). El principio general es *least privilege*: dar el mínimo acceso necesario, nunca "todo por las dudas".

## User Consent and Privacy

### User Notice

Una política de privacidad accesible y en lenguaje entendible (no un documento legal de 20 páginas que nadie lee) que explica qué datos se recolectan, para qué, y por cuánto tiempo se guardan. GDPR exige que esta información esté disponible **antes** de recolectar el dato, no escondida después del registro.

### Cookie Policy

Documentar qué cookies usa el sitio y por qué, separadas por categoría:

- **Esenciales**: necesarias para que el sitio funcione (ej. la cookie de sesión) — no requieren consentimiento.
- **No esenciales**: analytics, marketing, tracking de terceros — sí requieren consentimiento explícito antes de cargarse.

### User Consent

GDPR exige **opt-in**, no opt-out — el usuario tiene que aceptar activamente antes de que se activen cookies no esenciales, no verlas activas por default con la opción de rechazarlas después. Un banner de cookies que ya viene con todo tildado, o que sigue cargando scripts de analytics aunque el usuario todavía no respondió, no cumple con esto (patrón conocido como *dark pattern*).

```js
// las cookies/scripts no esenciales solo se cargan si hay consentimiento explícito guardado
function cargarAnalytics() {
  const consentimiento = localStorage.getItem('cookie-consent');
  if (consentimiento !== 'accepted') return; // sin consentimiento, no se carga nada

  const script = document.createElement('script');
  script.src = 'https://analytics.ejemplo.com/script.js';
  document.body.appendChild(script);
}
```

El consentimiento también tiene que ser tan fácil de **retirar** como de dar — un botón de "rechazar" al mismo nivel visual que "aceptar", no escondido en un submenú de configuración.

---
Relacionado: [Almacenamiento en el cliente](almacenamiento-cliente.md#cookies), [Accesibilidad](accesibilidad.md).
