# Qué es una base de datos

Un sistema diseñado específicamente para almacenar, organizar y recuperar datos de forma **persistente**, **concurrente** (varios procesos o usuarios accediendo a la vez sin pisarse) y **confiable** — a diferencia de guardar datos en un archivo de texto o JSON en disco.

## Por qué no alcanza con un archivo

```python
# ❌ un archivo JSON no resuelve nada de lo que sigue:
import json

def guardar_usuario(usuario):
    with open('usuarios.json') as f:
        data = json.load(f)
    data.append(usuario)
    with open('usuarios.json', 'w') as f:
        json.dump(data, f)

# si dos procesos llaman a esto al mismo tiempo, uno puede pisar el cambio
# del otro — no hay manejo de concurrencia, y si el proceso se cae justo
# en medio de la escritura, el archivo puede quedar corrupto a medias
```

Una base de datos resuelve, de fábrica, varios problemas que armar algo a mano con archivos no resuelve gratis:

- **Concurrencia**: múltiples clientes leyendo/escribiendo al mismo tiempo, sin pisarse — ver [Locks](locks.md).
- **Durabilidad**: si el proceso se cae a mitad de una escritura, el dato no queda corrupto — ver [Write-Ahead Log](rdbms.md#write-ahead-log-wal--cómo-no-se-pierde-nada-si-el-servidor-se-cae).
- **Consultas eficientes**: buscar, filtrar y ordenar sin tener que leer y parsear todo el archivo en memoria cada vez — ver [Índices](indices.md).
- **Integridad**: reglas que la propia base fuerza, para que un dato nunca quede en un estado a medias o inválido — ver [ACID](acid-transacciones-isolation.md).

## Los dos grandes tipos

- **Relacional (RDBMS)**: datos organizados en tablas con relaciones estrictas entre ellas, SQL como lenguaje de consulta — ver [RDBMS](rdbms.md).
- **NoSQL**: se aparta del modelo de tablas para priorizar escalabilidad horizontal o esquemas flexibles — ver [NoSQL](nosql.md).

---
Relacionado: [RDBMS](rdbms.md), [NoSQL](nosql.md).
