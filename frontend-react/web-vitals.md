# Web Vitals

Métricas concretas que Google define para medir "qué tan bien se siente" cargar y usar una página — reemplazan intuiciones vagas ("se siente lento") por números medibles, y son señal de ranking en el buscador. Las mide [Lighthouse](performance-diagnostics.md#lighthouse-genérico), entre otras herramientas.

## Core Web Vitals

- **LCP (Largest Contentful Paint)**: cuánto tarda en pintarse el elemento más grande visible en la pantalla inicial (una imagen hero, un bloque de texto grande). Es la métrica de "¿ya se siente cargado?" — un buen LCP es menor a 2.5s. Causas típicas de un LCP malo: imágenes sin optimizar, fuentes que bloquean el render, JS que retrasa cuándo aparece el contenido (ver [CSR vs SSR](renderizado.md#csr-vs-ssr)).
- **INP (Interaction to Next Paint)**: cuánto tarda la UI en responder visualmente después de que el usuario interactúa (click, tap, tecla) — reemplazó a FID (First Input Delay) porque mide la interacción **completa**, no solo el primer input. Un INP alto suele venir de JS bloqueando el hilo principal en el momento de la interacción (ver [Bloqueo del hilo principal](../diagnostico/frontend.md#bloqueo-del-hilo-principal)).
- **CLS (Cumulative Layout Shift)**: cuánto "salta" el layout mientras carga la página — imágenes o ads sin `width`/`height` reservado que empujan el contenido de abajo cuando terminan de cargar, un banner que aparece tarde y corre todo hacia abajo. Se mide sumando cuánto se movió cada elemento visible, ponderado por qué tan grande fue el salto.

```html
<!-- ❌ sin dimensiones reservadas: cuando la imagen carga, empuja el texto de abajo (CLS alto) -->
<img src="banner.jpg" />

<!-- ✅ el browser reserva el espacio desde el primer render, aunque la imagen tarde en llegar -->
<img src="banner.jpg" width="800" height="400" />
```

## Por qué importan más que "página carga rápido" en general

Un tiempo de carga total bajo no garantiza una buena experiencia si el contenido principal tarda en aparecer (LCP malo), si la página se siente trabada al tocarla (INP malo), o si el usuario termina haciendo click en el lugar equivocado porque algo se movió (CLS malo). Las tres miden momentos distintos de la experiencia — carga inicial, capacidad de respuesta, y estabilidad visual — por eso ninguna sola alcanza para decir "esta página anda bien".

---
Relacionado: [Performance Diagnostics](performance-diagnostics.md), [Diagnóstico Frontend](../diagnostico/frontend.md).
