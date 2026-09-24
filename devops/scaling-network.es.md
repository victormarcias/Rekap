# Escalabilidad de Red

Capacidad de red a nivel infraestructura — distinto del *chattiness* a nivel código (muchas llamadas chicas en vez de pocas grandes), que ya está cubierto en [HTTP chaining](../diagnostics/backend.es.md#http-chaining). Acá el problema no es cómo llama tu código, es que el **ancho de banda disponible** ya está saturado, sin importar qué tan eficiente sea la llamada.

## Cuándo el cuello de botella es la red, no el código

Se nota con CPU y memoria relajadas, pero throughput bajo y latencia alta — el proceso no está ocupado calculando, está esperando a que los datos terminen de viajar. Típico en transferencias grandes (backups, réplicas de DB, streaming de archivos) o en tráfico este-oeste alto entre muchos microservicios en la misma región.

## Escalar la capacidad de red

- **Instancias con más ancho de banda dedicado**: en la nube, el ancho de banda de red suele escalar junto con el tamaño de la instancia (una instancia más grande no solo tiene más CPU/RAM, también más Gbps disponibles) — a veces el "cuello de botella de CPU" real es en verdad un límite de red de una instancia chica.
- **CDN para tráfico saliente hacia usuarios**: sacar el tráfico de assets estáticos del origin server hacia el edge reduce directamente la carga de red que tiene que soportar tu infraestructura — ver [CDN](cdn.es.md).
- **Compresión**: menos bytes viajando por el mismo ancho de banda — `gzip`/`br` en las respuestas reduce la presión de red sin cambiar la infraestructura.
- **Acercar los servicios que hablan mucho entre sí**: dos servicios en distintas regiones que se llaman seguido pagan latencia de red alta en cada llamada — colocarlos en la misma región/zona de disponibilidad reduce esa latencia y el tráfico inter-región (que además suele facturarse aparte).

---
Relacionado: [CDN](cdn.es.md), [HTTP chaining](../diagnostics/backend.es.md#http-chaining), [Escalabilidad vertical vs horizontal](scaling-vertical-vs-horizontal.es.md).
