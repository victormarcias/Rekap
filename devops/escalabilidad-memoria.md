# Escalabilidad de Memoria

Capacidad de RAM a nivel infraestructura — distinto del memory leak a nivel código, que ya está cubierto en [Diagnóstico Backend](../diagnostics/backend.es.md#memoria). Acá el problema no es que el proceso pierda memoria con el tiempo, es que la carga de trabajo **legítima** ya necesita más RAM de la que tiene disponible.

## OOMKilled

En Kubernetes, si un pod supera el `limit` de memoria seteado en su [resource limits](kubernetes.md#resource-limits), el kernel lo mata inmediatamente (`OOMKilled`) — no hay degradación gradual, es un corte abrupto. Ver ese `OOMKilled` en los logs del pod es la señal de que el límite quedó chico para la carga real, no necesariamente de un leak.

```yaml
resources:
  limits: { memory: "512Mi" }  # si el proceso necesita más que esto, K8s lo mata, no lo throttlea
```

## Cuándo escalar memoria

- **Vertical**: subir el tamaño de instancia/el `limit` del pod, cuando el uso de memoria es genuinamente proporcional a la carga (más usuarios concurrentes, datasets más grandes en memoria).
- **Cachés en RAM que crecen sin límite**: un cache sin política de evicción (sin TTL, sin límite de tamaño) eventualmente consume toda la memoria disponible — no es un leak técnico, pero el síntoma es el mismo. Un store externo (ej. Redis) con políticas de evicción resuelve esto, en vez de cachear todo en memoria del proceso.
- **Swap**: cuando la RAM física se agota, el sistema operativo puede usar disco como memoria virtual — funciona, pero el acceso a disco es órdenes de magnitud más lento que RAM; un proceso "swappeando" activamente tiene toda la pinta de estar colgado aunque técnicamente siga vivo.

---
Relacionado: [Diagnóstico Backend](../diagnostics/backend.es.md#memoria) (memory leaks), [Kubernetes](kubernetes.md#resource-limits).
