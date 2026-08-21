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

## Breve historia de las bases de datos

Video: [Breve historia de las bases de datos](https://www.youtube.com/watch?v=KG-mqHoXOXY)

Antes de que el modelo relacional se impusiera, existieron (y en algunos nichos, todavía existen) otros modelos de datos:

- **Jerárquico** (ej. IBM IMS, 1966; el registro de Windows): los datos se organizan en árbol, cada registro tiene un solo padre. Simple, pero rígido — modelar una relación muchos-a-muchos es forzado.
- **De red** (CODASYL): como el jerárquico, pero un registro puede tener varios padres — más flexible, a costa de que navegar los datos requería conocer la estructura física de antemano. Es, conceptualmente, el antecesor directo de lo que hoy es una base de datos de grafo.
- **Relacional**: el que ganó — tablas independientes de cómo se accede a ellas después, consultadas con SQL declarativo en vez de navegación manual entre registros. Ver [RDBMS](rdbms.md).
- **Orientado a objetos** (ej. db4o, ObjectDB): guarda objetos tal como existen en memoria en un lenguaje OOP, sin traducirlos a tablas. Nunca despegó fuera de nichos puntuales — un ORM resuelve el mismo problema (persistir objetos) pero traduciéndolos a tablas relacionales por debajo, en vez de evitar esa traducción.
- **Plano** (Flat file): un solo archivo, sin relaciones — un CSV es, en esencia, esto.
- **Semi-estructurado**: JSON/XML sin schema fijo — es lo que hoy cubren las bases de datos NoSQL de tipo documento. Ver [NoSQL](nosql.md).

**Entity-Relationship (E-R) no es un modelo de almacenamiento** como los de arriba — es una técnica de **diseño** (los diagramas E-R) para modelar entidades y relaciones antes de traducirlas a tablas de un modelo relacional. Un ORM no trabaja "sobre E-R" directamente: trabaja sobre el resultado de ese diseño (las tablas relacionales), el diagrama E-R es un paso previo en la cabeza de quien diseña el schema.

## Los dos grandes tipos

- **Relacional (RDBMS)**: datos organizados en tablas con relaciones estrictas entre ellas, SQL como lenguaje de consulta — ver [RDBMS](rdbms.md).
- **NoSQL**: se aparta del modelo de tablas para priorizar escalabilidad horizontal o esquemas flexibles — ver [NoSQL](nosql.md).

---
Relacionado: [RDBMS](rdbms.md), [NoSQL](nosql.md).
