# Escalabilidad de Procesos

Sumar más procesos/workers en paralelo — el escalón intermedio entre "un solo proceso" y "escalar a múltiples máquinas". Ya vimos el patrón dentro de una sola máquina como [Clustering](../diagnostics/backend.es.md#clustering); esto es la misma idea, llevada al nivel de infraestructura completa.

## Dentro de una máquina, primero

Antes de sumar máquinas, aprovechar los cores que ya tenés: un runtime single-threaded (Node) corriendo un solo proceso deja el resto de los cores de la máquina ociosos. Correr un proceso por core (`cluster` module, PM2 en modo cluster, múltiples workers de gunicorn) es la forma más barata de escalar — no hay red de por medio, solo procesos separados en la misma máquina.

## Entre máquinas, después

Una vez que una sola máquina no da más abasto (todos sus cores ya están usados, y sigue faltando capacidad), el siguiente paso es [escalar horizontalmente](escalabilidad-vertical-horizontal.md) sumando más instancias/máquinas — cada una corriendo su propio set de procesos — repartidas por un [load balancer](../backend/load-balancers.md).

## El requisito: procesos sin estado propio

Ya sea multi-proceso en una máquina o multi-instancia entre máquinas, cada proceso tiene que poder atender cualquier request sin depender de estado que solo él tiene en memoria — la misma condición de [Escalabilidad](../system-design/atributos-de-calidad.md#escalabilidad) stateless que habilita escalar horizontalmente sin romper nada. Un proceso que guarda sesiones en su propia memoria rompe apenas hay más de uno atendiendo el mismo tráfico.

---
Relacionado: [Clustering](../diagnostics/backend.es.md#clustering), [Escalabilidad vertical vs horizontal](escalabilidad-vertical-horizontal.md), [Load balancers](../backend/load-balancers.md).
