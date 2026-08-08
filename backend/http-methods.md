# HTTP Methods

Los verbos HTTP tienen semántica definida por el protocolo — usarlos "porque sí" (todo `POST`) tira esa información a la basura.

## Los verbos y su propósito

- **GET**: leer un recurso. No debería tener efectos secundarios.
- **POST**: crear un recurso nuevo, o ejecutar una acción que no encaja en los demás verbos.
- **PUT**: reemplazar un recurso **completo** — el body manda el objeto entero, lo que no se incluye se pierde/resetea.
- **PATCH**: actualizar **parcialmente** un recurso — el body manda solo los campos que cambian.
- **DELETE**: borrar un recurso.

```
PUT /users/1    {"name": "Ana", "email": "ana@mail.com"}
  → reemplaza el usuario entero; si tenía un campo "phone" y no lo mandaste, se pierde

PATCH /users/1  {"email": "nueva@mail.com"}
  → solo actualiza el email, el resto del usuario queda intacto
```

## Safe methods — sin efectos secundarios

**GET**, **HEAD**, **OPTIONS** son *safe*: no deberían cambiar el estado del servidor. Esto no es solo una convención — un browser, un crawler, o un proxy pueden reintentar un `GET` libremente (para cache, prefetch, etc.) asumiendo que no importa cuántas veces se ejecute. Un endpoint `GET` que borra datos rompe esa garantía y puede causar borrados accidentales por herramientas que no esperan que un `GET` tenga efectos.

## Idempotencia por verbo

Ya cubierto en detalle en [Idempotencia](../system-design/atributos-de-calidad.md#idempotencia) — repaso rápido aplicado a los verbos:

| Verbo | Idempotente | Por qué |
|---|---|---|
| GET | Sí | Leer no cambia nada, sin importar cuántas veces |
| PUT | Sí | Reemplazar por el mismo valor N veces da el mismo resultado que una vez |
| DELETE | Sí | Borrar algo que ya no existe sigue dando "no existe" |
| PATCH | Depende | Si el patch es `{"stock": 5}` sí; si es `{"stock": stock - 1}` no — cada aplicación resta de nuevo |
| POST | No | Cada `POST` crea un recurso nuevo — reintentar sin idempotency key duplica |

Esto es exactamente por qué un reintento automático de red es seguro en un `PUT`/`DELETE` pero riesgoso en un `POST` sin una idempotency key.

---
Relacionado: [REST](rest.md) (los métodos son una pieza de su interfaz uniforme, no lo mismo que REST), [Idempotencia](../system-design/atributos-de-calidad.md#idempotencia), [HTTP Status Codes](../system-design/http-status-codes.md) (`405 Method Not Allowed` cuando un recurso no soporta el verbo pedido).
