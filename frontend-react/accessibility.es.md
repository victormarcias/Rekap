# Accesibilidad

## Qué es accesibilidad

Que una persona pueda usar la app sin importar cómo interactúa con ella — con mouse, solo con teclado, con un screen reader, con zoom alto, con daltonismo, con movilidad reducida. No es una feature aparte que se agrega al final; es una propiedad de cómo se construyó el HTML/CSS/JS desde el principio (ver [HTML](html.es.md) — la semántica correcta ya es la mitad del trabajo).

## Universal Design vs Accessible Design

- **Accessible Design**: adaptar algo existente para que también lo puedan usar personas con discapacidad — a menudo una solución paralela o un parche (ej. una versión "solo texto" de un sitio).
- **Universal Design**: diseñar desde el principio para que funcione para todos, sin necesitar una versión paralela — un subtítulo ayuda a alguien sordo, pero también a alguien mirando el video sin sonido en el colectivo. Los principios de accesibilidad terminan siendo buen diseño para todo el mundo, no una concesión.

## Tipos de problemas de accesibilidad

- **Visual**: bajo contraste de color, texto que no escala con zoom, información transmitida solo por color (ej. "los campos en rojo son obligatorios" sin ningún otro indicador).
- **Motora**: elementos interactivos muy chicos o muy juntos (difícil de tocar con precisión), funcionalidad que solo responde a hover o drag sin alternativa de teclado.
- **Auditiva**: video/audio sin subtítulos ni transcripción.
- **Cognitiva**: lenguaje innecesariamente complejo, layouts inconsistentes entre páginas, tiempos límite muy cortos para completar una acción (ej. una sesión que expira sin aviso).

## Screen readers

Software que lee el contenido de la pantalla en voz alta (o en braille), navegando por la estructura semántica del HTML — no por cómo se ve, sino por qué **es** cada elemento (`<button>` anuncia "botón", `<nav>` anuncia "navegación"). Por eso el HTML semántico y los atributos ARIA (`aria-label`, `aria-expanded`, `role`) no son decoración: son literalmente lo único que el screen reader tiene para describir la interfaz.

```html
<!-- sin aria-label, el screen reader solo anuncia "botón" — sin decir para qué -->
<button aria-label="Cerrar modal">✕</button>

<!-- aria-expanded le avisa al screen reader el estado de un elemento colapsable,
     algo que un usuario vidente ve por el ícono de flecha -->
<button aria-expanded="false" aria-controls="menu">Menú</button>
```

## Alt texts

El atributo `alt` de una imagen es lo que anuncia un screen reader en lugar de la imagen, y lo que se muestra si la imagen no carga. Un `alt` vacío (`alt=""`) es una decisión válida y explícita para imágenes puramente decorativas — le dice al screen reader "salteame, no aporta información" — muy distinto de no poner `alt` en absoluto, que hace que algunos screen readers lean el nombre del archivo (`IMG_4821.jpg`).

```html
<!-- ✅ describe la información que la imagen aporta -->
<img src="grafico-ventas.png" alt="Ventas creciendo 30% en el último trimestre" />

<!-- ✅ decorativa, no aporta información — se salta explícitamente -->
<img src="linea-decorativa.svg" alt="" />

<!-- ❌ no describe nada útil -->
<img src="grafico-ventas.png" alt="imagen" />
```

---
Relacionado: [HTML](html.es.md).
