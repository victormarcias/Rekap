# CSS

## CSS Reset / Normalize

Cada browser trae sus propios estilos por default para los elementos HTML (márgenes en `<body>`, `<ul>` con bullets y padding, tamaños de `<h1>`...`<h6>` distintos, etc.) — y no son exactamente iguales entre Chrome, Firefox y Safari. Un reset es una hoja de estilos que se carga primero, antes que cualquier CSS propio, para partir de una base predecible.

```css
/* Reset "duro" (ej. estilo Eric Meyer): borra todo, incluso lo útil —
   después hay que redefinir tamaños de heading, listas, etc. a mano */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
ul, ol { list-style: none; }
a { text-decoration: none; color: inherit; }
```

**Normalize.css** es el enfoque más moderno: en vez de borrar todo, **corrige las inconsistencias entre browsers** manteniendo los defaults que sí son útiles (un `<h1>` sigue siendo más grande que el texto normal, pero el tamaño exacto queda igualado entre browsers). La mayoría de los proyectos actuales usan esto en vez de un reset duro — o ni siquiera lo agregan a mano: frameworks como Tailwind ya traen su propio reset moderno (el "preflight") incluido.

Por qué importa: sin esto, el mismo HTML/CSS puede verse ligeramente distinto según el browser sin que haya ningún bug en el código — solo defaults distintos compitiendo con los estilos propios.

## Especificidad y selectores

El browser resuelve conflictos entre reglas CSS por **especificidad**, no por quién "se ve más específico" a simple vista. De menor a mayor peso: selector de elemento (`div`) < clase/atributo/pseudo-clase (`.card`, `[type="text"]`, `:hover`) < id (`#header`) < estilos inline (`style="..."`) < `!important`.

```css
div.card { color: blue; }      /* especificidad: 0-1-1 */
#header .card { color: red; }  /* especificidad: 1-1-0 → gana esta */
```

`!important` rompe la cascada normal y gana casi siempre — por eso es una señal de alarma en un codebase: suele indicar que alguien no pudo (o no supo cómo) ganarle a otra regla de forma prolija, y termina generando una carrera de `!important` contra `!important`.

### `@layer` — ordenar la cascada sin pelear con especificidad

`@layer` (CSS Cascade Layers) declara explícitamente el orden de prioridad entre grupos de reglas — una capa declarada después le gana a una declarada antes, **sin importar la especificidad de cada regla individual dentro de esa capa**. Es la forma moderna de evitar la guerra de `!important` cuando conviven varias librerías (Bootstrap, un design system, tu propio CSS) que compiten por los mismos elementos.

```css
/* el orden de esta línea define la prioridad — de menor a mayor */
@layer reset, libraries, components, utilities;

@layer libraries {
  @import url("bootstrap.css"); /* lo que traiga Bootstrap queda contenido en esta capa */
}

@layer components {
  /* le gana a CUALQUIER regla de .libraries, aunque esa regla tenga
     mayor especificidad (ej. un id) — la capa manda antes que la especificidad */
  .boton { color: blue; }
}
```

Dentro de una misma capa, la especificidad normal sigue aplicando — `@layer` no reemplaza esas reglas, agrega un nivel de prioridad **por encima** de ellas. Cualquier CSS que no esté dentro de ningún `@layer` se trata como la capa de mayor prioridad de todas, así que el código sin capas de siempre sigue ganando por default sin tener que migrarlo.

## Flexbox vs Grid

- **Flexbox**: layout en **una dimensión** — una fila o una columna. Pensado para distribuir espacio entre ítems de una lista o alinear elementos dentro de un contenedor.
- **Grid**: layout en **dos dimensiones** — filas y columnas a la vez. Pensado para la estructura general de una página o de un componente complejo (dashboard, galería).

```css
/* Flex: una barra de navegación en fila, separada a los extremos */
.navbar { display: flex; justify-content: space-between; align-items: center; }

/* Grid: layout de página con sidebar fija y contenido flexible */
.layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
}
```

No compiten — es común usar Grid para la estructura general y Flex adentro de cada celda del grid.

## CSS Variables (Custom Properties)

Valores reutilizables definidos con `--nombre` y leídos con `var()`. A diferencia de las variables de un preprocesador (SASS), viven en el DOM en tiempo de ejecución — se pueden leer/cambiar con JS y respetan la cascada normal (se pueden sobreescribir por selector, útil para temas claro/oscuro).

```css
:root { --color-primario: #6b21a8; --spacing: 8px; }

.boton { background: var(--color-primario); padding: calc(var(--spacing) * 2); }

/* tema oscuro: mismo nombre de variable, valor distinto en un scope distinto */
[data-theme="dark"] { --color-primario: #a78bfa; }
```

## Animations vs Transitions

- **Transition**: interpola entre dos estados cuando algo cambia (hover, foco, una clase que se agrega/saca). Necesita un disparador — no arranca sola.
- **Animation**: define una secuencia con `@keyframes`, puede tener múltiples pasos intermedios, loopear, y arrancar sola al cargar el elemento — no depende de que algo cambie.

```css
/* Transition: solo interpola entre el estado normal y :hover */
.boton { transition: transform 0.2s ease; }
.boton:hover { transform: scale(1.05); }

/* Animation: secuencia de varios pasos, se repite sola */
@keyframes pulso {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}
.spinner { animation: pulso 1.5s infinite; }
```

## Stacking contexts (por qué mi `z-index` no funciona)

`z-index` no compite globalmente contra todos los `z-index` de la página — solo compite **dentro del mismo stacking context**. Un elemento crea su propio stacking context nuevo (aislando a sus hijos del resto de la página) cuando tiene, entre otras cosas: `position` distinto de `static` + un `z-index` explícito, `opacity` menor a 1, `transform`, o `will-change`.

```css
/* el modal tiene z-index: 9999, pero si su contenedor padre
   tiene opacity < 1 o un transform, el modal queda "encerrado"
   en el stacking context del padre y no puede superar a elementos
   que están afuera de ese padre, sin importar cuán alto sea su z-index */
.padre { opacity: 0.99; } /* crea un stacking context sin querer */
.modal { position: fixed; z-index: 9999; } /* atrapado adentro del padre */
```

Este es el bug clásico: subir el `z-index` cada vez más alto no soluciona nada si el problema es que el elemento está atrapado en el stacking context equivocado — hay que resolverlo en el padre (sacar el `opacity`/`transform` que no hace falta, o usar un [portal](https://react.dev/reference/react-dom/createPortal) en React para renderizar el modal fuera del árbol del padre).

## Metodologías para organizar y scopear estilos

Todas resuelven el mismo problema — "cómo evito que mis clases choquen entre componentes" — desde capas distintas:

- **BEM** (`Block__Element--Modifier`): convención de nombres en CSS plano, sin build tool. Evita colisiones por disciplina, no por herramienta.
  ```css
  .card { }
  .card__title { }
  .card--featured { }
  ```
- **CSS Modules**: cada `.module.css` se procesa en build time y sus clases se renombran automáticamente a algo único (`.card_a3f1x`) — el scoping lo garantiza la herramienta, no la convención.
  ```jsx
  import styles from './Card.module.css';
  <div className={styles.card}>...</div>
  ```
- **CSS-in-JS** (ej. styled-components): los estilos se escriben en JS, colocados junto al componente, y pueden depender de props en runtime — pero eso tiene un costo de performance (generar CSS en el cliente) que CSS Modules no tiene.
  ```jsx
  const Card = styled.div`
    background: ${props => props.featured ? '#fef3c7' : 'white'};
  `;
  ```
- **SASS/SCSS**: preprocesador — agrega variables, nesting y mixins que compilan a CSS plano antes de llegar al browser. No resuelve scoping por sí solo (para eso se suele combinar con BEM o CSS Modules), resuelve la falta de reutilización de CSS plano.
  ```scss
  $color-primario: #6b21a8;
  .card {
    &__title { color: $color-primario; }
    &:hover { opacity: 0.9; }
  }
  ```
