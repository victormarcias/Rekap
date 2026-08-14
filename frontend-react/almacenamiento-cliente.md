# Almacenamiento en el cliente

Dónde vive el estado de una app y por cuánto tiempo — la respuesta cambia según qué tan sensible es el dato y si necesita sobrevivir a un refresh, un cierre de pestaña, o un logout.

## Estado en memoria (JavaScript)

Una variable o un `useState` normal — vive mientras la página esté cargada, desaparece al refrescar. Es el lugar por defecto para cualquier estado de UI que no necesita persistir (un modal abierto, un input sin submitear).

## `localStorage`

Persiste indefinidamente en el browser, sobrevive a cerrar la pestaña y a reiniciar la compu, con scope por origen (protocolo + dominio + puerto). Solo guarda strings — objetos necesitan `JSON.stringify`/`JSON.parse`.

```js
localStorage.setItem('theme', 'dark');
const theme = localStorage.getItem('theme'); // 'dark'
localStorage.setItem('user', JSON.stringify({ id: 1, name: 'Vic' }));
```

## `sessionStorage`

Misma API que `localStorage`, pero el scope es la **pestaña**: se borra al cerrarla, y no se comparte entre pestañas del mismo sitio (cada pestaña tiene su propio `sessionStorage`, aunque abran la misma URL).

## Cookies

A diferencia de la creencia popular, **cookies es la forma más segura de guardar información en el cliente** — no `localStorage`. La razón es un flag que ni `localStorage` ni `sessionStorage` pueden tener: `HttpOnly`.

```
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

- **`HttpOnly`**: la cookie es invisible para JavaScript (`document.cookie` no la muestra) — si la app tiene una vulnerabilidad XSS, el script malicioso inyectado puede leer todo lo que haya en `localStorage`, pero no una cookie `HttpOnly`. Este es el motivo de fondo: `localStorage` es 100% accesible por cualquier JS que corra en la página, cookies pueden no serlo.
- **`Secure`**: solo se manda por HTTPS, nunca en texto plano.
- **`SameSite`**: controla si la cookie se manda en requests que vienen de otro sitio — `Strict`/`Lax` mitigan CSRF.

El trade-off: cookies se mandan automáticamente en **cada** request al mismo dominio (suman peso a cada request) y tienen un límite de tamaño chico (~4KB) — por eso no sirven para guardar blobs grandes de datos, solo identificadores como un session token.

## Estado en el servidor

El dato más sensible o compartido entre dispositivos no vive en el cliente en absoluto — el cliente solo guarda un identificador (ej. una cookie de sesión o un JWT), y el servidor resuelve ese identificador contra su propio storage (DB, Redis) para saber quién es el usuario y qué le corresponde. Es la única opción cuando el estado tiene que ser consistente entre el celular y la notebook del mismo usuario al mismo tiempo.

## Cuál usar

| | Persiste refresh | Persiste cerrar pestaña | Accesible por JS | Va en cada request | Límite de tamaño |
|---|---|---|---|---|---|
| Memoria (`useState`) | ❌ | ❌ | ✅ | ❌ | Limitado por la memoria RAM disponible — en la práctica, sin límite fijo |
| `sessionStorage` | ✅ | ❌ | ✅ | ❌ | ~5-10MB por origen (varía por browser) |
| `localStorage` | ✅ | ✅ | ✅ | ❌ | ~5-10MB por origen (varía por browser) |
| Cookie (`HttpOnly`) | ✅ | ✅ | ❌ | ✅ | ~4KB por cookie |

Regla práctica: tokens de sesión / auth → cookie `HttpOnly`. Preferencias de UI sin nada sensible (tema, idioma) → `localStorage`. Estado de un flujo de varios pasos en la misma visita (ej. un wizard) → `sessionStorage`. Cualquier cosa que dependa de otros usuarios o deba ser la fuente de verdad → servidor.
