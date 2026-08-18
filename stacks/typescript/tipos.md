# TypeScript — Sistema de tipos

## Tipado de funciones

Cada parámetro y el valor de retorno pueden tipar explícitamente — el compilador rechaza una llamada que no cumpla el contrato antes de que el código llegue a correr.

```ts
function calcularTotal(precio: number, cantidad: number): number {
  return precio * cantidad;
}

calcularTotal(10, "2"); // ❌ error en compile time: Argument of type 'string' is not assignable to parameter of type 'number'

// parámetros opcionales (?) y default values
function saludar(nombre: string, saludo: string = "Hola"): string {
  return `${saludo}, ${nombre}`;
}

// funciones como valores: el tipo describe la firma completa
const sumar: (a: number, b: number) => number = (a, b) => a + b;
```

Por qué importa: sin esto, un error de tipo (pasar un string donde se espera un number) recién se descubre en runtime, potencialmente en producción — con tipado, el error aparece en el editor mientras se escribe el código.

## Type Aliases

Nombrar un tipo con `type` para reutilizarlo sin repetir la forma completa cada vez — no crea un tipo nuevo, es solo un nombre legible para algo que ya se podría escribir inline.

```ts
// ❌ sin alias: repetir la misma forma en cada función que la necesita
function actualizar(usuario: { id: string; nombre: string; email: string }) { }
function crear(usuario: { id: string; nombre: string; email: string }) { }

// ✅ con alias: se define una vez, se reutiliza
type Usuario = { id: string; nombre: string; email: string };
function actualizar(usuario: Usuario) { }
function crear(usuario: Usuario) { }
```

**`type` vs `interface`**: para describir la forma de un objeto se solapan bastante. La diferencia práctica: `type` puede nombrar cualquier cosa (uniones, tuplas, primitivos), mientras que `interface` solo describe objetos, pero se puede extender/reabrir en varios lugares del código (útil en librerías). Regla simple: `interface` para la forma de un objeto que puede crecer con el tiempo, `type` para todo lo demás.

## Unions e Intersections

- **Union (`|`)**: el valor puede ser **uno entre varios** tipos posibles.
- **Intersection (`&`)**: el valor debe cumplir **todos** los tipos combinados a la vez.

```ts
// Union: el estado de un request solo puede ser uno de estos tres strings literales
type EstadoRequest = "idle" | "loading" | "error" | "success";

function manejarEstado(estado: EstadoRequest) {
  if (estado === "loading") { /* ... */ }
  // TypeScript sabe, dentro de este if, que estado no puede ser otro valor —
  // autocompletado y chequeo exhaustivo de los casos posibles
}

// Intersection: combina las props de dos tipos en uno solo
type ConId = { id: string };
type ConNombre = { nombre: string };
type Usuario = ConId & ConNombre; // { id: string; nombre: string }
```

**Narrowing**: dentro de un `if (typeof x === "string")` o un `if ("prop" in obj)`, TypeScript "achica" automáticamente el tipo union a la opción que aplica en ese bloque — no hace falta castear manualmente.

## Nullable Types

TypeScript no tiene un tipo "opcional" separado — la ausencia de valor se modela con una union explícita contra `null` y/o `undefined`.

```ts
function buscarUsuario(id: string): Usuario | undefined {
  // devuelve undefined si no lo encuentra
}

const usuario = buscarUsuario("123");
usuario.nombre;   // ❌ error: 'usuario' is possibly 'undefined'
usuario?.nombre;   // ✅ optional chaining — accede solo si no es null/undefined
usuario!.nombre;   // ⚠️ non-null assertion — "confiá en mí, no es null" — sin chequeo real, riesgo de error en runtime si te equivocás
```

Con [`strict`](config.md#strict) prendido, TypeScript obliga a manejar el caso `null`/`undefined` explícitamente antes de usar el valor — sin `strict`, deja pasar el acceso directo y el error solo aparece en runtime.

## `unknown`

El tipo seguro para "todavía no sé qué es esto" — a diferencia de `any`, que apaga el chequeo de tipos por completo, `unknown` obliga a **verificar el tipo antes de poder usarlo**.

```ts
function procesar(valor: unknown) {
  valor.toUpperCase();          // ❌ error: 'valor' is of type 'unknown'

  if (typeof valor === "string") {
    valor.toUpperCase();         // ✅ ya narrowed a string dentro de este if
  }
}

function procesarAny(valor: any) {
  valor.toUpperCase();          // ✅ compila igual, aunque valor sea un número — sin protección real
}
```

`unknown` es la opción correcta cuando el tipo realmente no se sabe de antemano (ej. la respuesta cruda de una API externa) — `any` debería ser la excepción, no el default.

## `never`

El tipo de algo que **nunca produce un valor** — una función que siempre lanza una excepción, o una rama de código matemáticamente imposible de alcanzar.

```ts
function fallar(mensaje: string): never {
  throw new Error(mensaje); // nunca retorna, siempre lanza
}

// uso típico: chequeo exhaustivo de un union — si se agrega un caso nuevo
// y no se maneja acá, TypeScript marca error en tiempo de compilación
type Estado = "idle" | "loading" | "error";

function manejar(estado: Estado) {
  switch (estado) {
    case "idle": return "esperando";
    case "loading": return "cargando";
    case "error": return "falló";
    default:
      const _exhaustivo: never = estado; // si falta un caso, esta línea no compila
      return _exhaustivo;
  }
}
```

## Utility Types

Tipos genéricos que vienen con TypeScript para transformar un tipo existente sin reescribirlo a mano — evitan duplicar la forma de un objeto para cada variante que se necesita (ej. "todos los campos opcionales" o "solo estos dos campos").

```ts
type Usuario = { id: string; nombre: string; email: string };

type UsuarioParcial = Partial<Usuario>;  // { id?: string; nombre?: string; email?: string }
type SoloContacto = Pick<Usuario, "nombre" | "email">; // { nombre: string; email: string }
type SinEmail = Omit<Usuario, "email">;  // { id: string; nombre: string }
type UsuariosPorId = Record<string, Usuario>; // { [key: string]: Usuario }
```

`Partial` es el más común en la práctica: una función de "update" suele recibir solo los campos que cambian, no el objeto completo.

## Generics (tipos avanzados)

Un tipo "parametrizado" — en vez de fijar de antemano con qué tipo trabaja una función o estructura, se define una vez y se instancia con el tipo real en cada uso, sin perder el chequeo de tipos (a diferencia de usar `any`, que apaga el chequeo por completo).

```ts
// sin generics: hay que repetir la función para cada tipo, o usar any y perder seguridad
function primero<T>(lista: T[]): T {
  return lista[0];
}

primero([1, 2, 3]);       // tipo inferido: number
primero(["a", "b"]);      // tipo inferido: string

// generics en una interfaz — un Response que puede envolver cualquier tipo de dato
interface ApiResponse<T> {
  data: T;
  error: string | null;
}

const respuesta: ApiResponse<Usuario> = { data: usuario, error: null };
```

**Conditional types** (`T extends U ? X : Y`) y **mapped types** (`{ [K in keyof T]: ... }`, la base de cómo están implementados los Utility Types de arriba) son la forma de escribir lógica a nivel de tipos — se usan mucho al construir librerías, menos seguido en código de aplicación día a día.

---
Relacionado: [Configuración (tsconfig.json)](config.md), [Sistema de tipos comparado (Python/Swift/Kotlin/Java)](../tipos-comparativa.md), [Frontend React](../../frontend-react/) para dónde se aplica esto en componentes.
