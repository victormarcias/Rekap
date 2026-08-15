# Clean Architecture

Propuesta por Robert C. Martin ("Uncle Bob"). Resuelve un problema concreto: la lógica de negocio de una app normalmente vive mezclada con detalles de infraestructura (el framework web, la base de datos, la UI) — y esos detalles cambian mucho más seguido que las reglas del negocio en sí. Clean Architecture separa ambas cosas para que un cambio de infraestructura (migrar de Postgres a Mongo, de Flask a FastAPI) no obligue a tocar ni entender la lógica de negocio.

## La Regla de Dependencia

El código fuente solo puede depender **hacia adentro** — las capas externas (framework, DB, UI) conocen y dependen de las internas (reglas de negocio); las internas **no saben que las externas existen**. La lógica de negocio no importa nada de FastAPI, de SQLAlchemy, ni de React — no tiene forma de saber con qué está corriendo por afuera.

## Las capas, de adentro hacia afuera

1. **Entities**: las reglas de negocio más generales, independientes de esta aplicación puntual — conceptos que existirían aunque cambiara todo el resto (ej. "un pedido enviado no se puede cancelar", una regla del dominio, no de esta app en particular).
2. **Use Cases**: reglas de negocio específicas de esta aplicación — orquestan las entities para lograr algo concreto (ej. "cancelar un pedido": buscarlo, validar que se pueda, guardarlo).
3. **Interface Adapters**: traducen datos entre el formato que usan los use cases y el formato que necesitan las capas externas — controllers (reciben el request HTTP y llaman al use case), presenters, gateways.
4. **Frameworks & Drivers**: la capa más externa y más reemplazable — el framework web, el driver de la base de datos, la UI. Acá sí "se sabe" que existe Postgres o FastAPI.

## Ejemplo — Dependency Inversion en la práctica

```python
# --- Entities: no importa nada de fuera, ni siquiera sabe que existe una DB ---
class Order:
    def __init__(self, id, total, status):
        self.id = id
        self.total = total
        self.status = status

    def cancel(self):
        if self.status == "shipped":
            raise ValueError("No se puede cancelar un pedido ya enviado")
        self.status = "cancelled"

# --- Use Case: define QUÉ necesita (una interfaz), no CÓMO se implementa ---
from typing import Protocol

class OrderRepository(Protocol):
    def get(self, order_id: str) -> Order: ...
    def save(self, order: Order) -> None: ...

class CancelOrderUseCase:
    def __init__(self, repo: OrderRepository):
        self.repo = repo  # depende de la ABSTRACCIÓN, no de Postgres/Mongo/lo que sea

    def execute(self, order_id: str):
        order = self.repo.get(order_id)
        order.cancel()
        self.repo.save(order)

# --- Frameworks & Drivers: acá SÍ se sabe que existe Postgres ---
class PostgresOrderRepository:  # implementa el "contrato" OrderRepository
    def get(self, order_id):
        ...  # query real a Postgres
    def save(self, order):
        ...  # UPDATE real a Postgres
```

`CancelOrderUseCase` nunca importa `psycopg2` ni `sqlalchemy` — solo conoce el contrato `OrderRepository`. Dos consecuencias directas: se puede testear con un `FakeOrderRepository` en memoria sin levantar una base real, y se puede migrar de Postgres a Mongo escribiendo un nuevo adapter, sin tocar una sola línea de la regla de negocio.

## Por qué importa

- **Testeable sin infraestructura real**: los tests del use case corren en milisegundos, sin DB ni red — el equivalente al [Fake de los test doubles](../system-design/testing.md#2-test-doubles--mock-vs-stub-vs-fake-vs-spy).
- **Independiente de framework y de DB**: cambiar de herramienta externa no debería obligar a reescribir la lógica de negocio.
- **El dominio se lee solo**: alguien nuevo en el equipo puede entender las reglas de negocio leyendo `Order`/`CancelOrderUseCase`, sin tener que entender FastAPI ni el ORM primero.

## Relación con Hexagonal Architecture (Ports & Adapters)

Mismo espíritu, distinta terminología — en la práctica se usan casi como sinónimos. Hexagonal habla de **ports** (las interfaces, como `OrderRepository` arriba) y **adapters** (las implementaciones concretas, como `PostgresOrderRepository`); Clean Architecture habla de capas concéntricas. El mecanismo de fondo es el mismo: el dominio define el contrato, la infraestructura lo implementa, nunca al revés.

**Plugin Architecture (Microkernel Architecture)** es un patrón hermano — mismo mecanismo (un core que define un contrato, y módulos externos que lo implementan para "enchufarse"), pero con otro énfasis: Hexagonal aísla el dominio de la **infraestructura técnica** ("puedo cambiar de Postgres a Mongo sin tocar el negocio"); Plugin/Microkernel extiende un core mínimo con **features opcionales** ("puedo agregar o sacar un módulo sin tocar el core") — el ejemplo típico son las extensiones de un IDE o los plugins de un CMS.

El **Dependency Inversion Principle** (la D de SOLID) es literalmente el mecanismo que hace posible la Regla de Dependencia — Clean Architecture es, en buena medida, DIP aplicado sistemáticamente a toda la app. [Controller / Service / Repository](../backend/controller-service-repository.md) es una versión más simple y pragmática del mismo espíritu — muchos equipos la usan sin llegar a implementar las 4 capas completas. Sirve como punto intermedio: menos ceremonia, buena parte del beneficio.

## Trade-off

No es gratis: agrega indirección (interfaces, mapeo de datos entre capas) que para un CRUD simple puede ser sobre-ingeniería — más archivos, más saltos para seguir el flujo de una operación. Vale la pena cuando la lógica de negocio es compleja y va a vivir mucho tiempo, o cuando realmente se espera cambiar de infraestructura en el futuro. Para un prototipo, un script, o un servicio muy simple, el costo de la indirección suele superar el beneficio.

---
Relacionado: [SOLID principles](solid.md), [Controller / Service / Repository](../backend/controller-service-repository.md), [Patrones estructurales](patrones-estructurales.md#adapter).
