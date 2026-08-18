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
Relacionado: [Configuración (tsconfig.json)](config.md), [Frontend React](../../frontend-react/) para dónde se aplica esto en componentes.
