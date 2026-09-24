# Triggers y Vistas

## Vistas (Views)

Una vista es una **query guardada con nombre** — se consulta como si fuera una tabla, pero no almacena datos propios: cada vez que se hace `SELECT` sobre ella, la DB ejecuta la query original por debajo, contra los datos reales y actuales de las tablas de base.

```sql
CREATE VIEW pedidos_pendientes AS
  SELECT o.id, o.customer_id, o.total, c.name AS customer_name
  FROM orders o
  JOIN customers c ON c.id = o.customer_id
  WHERE o.status = 'pending';

-- se consulta como una tabla normal
SELECT * FROM pedidos_pendientes WHERE total > 1000;
```

**Para qué sirve**: simplificar queries complejas/repetidas (esconder un JOIN de 4 tablas atrás de un nombre simple), y como capa de seguridad — dar acceso a una vista que solo expone ciertas columnas/filas, sin dar acceso a la tabla completa.

**El límite**: como no persiste nada, una vista sobre una query pesada (agregaciones, joins grandes) es **tan lenta como la query original**, cada vez que se consulta — no acelera nada por sí sola. Para eso existe la vista materializada.

## Vista vs Vista materializada

Ya cubierto en detalle en [Vistas materializadas](../diagnostics/database.es.md#vistas-materializadas) y comparado contra snapshot tables en [Snapshot tables vs vistas materializadas](escalabilidad-db.md#snapshot-tables-vs-vistas-materializadas) — acá el resumen de la diferencia de fondo:

| | Vista | Vista materializada |
|---|---|---|
| Almacena datos | No — ejecuta la query en cada consulta | Sí — persiste el resultado |
| Performance de lectura | Igual que la query original | Rápida — lee un resultado ya calculado |
| Actualidad del dato | Siempre al segundo | Desactualizada hasta el próximo refresh |
| Costo | Ninguno extra (no ocupa espacio) | Espacio de almacenamiento + costo de refrescar |

## Triggers

Código que la base de datos ejecuta **automáticamente** cuando ocurre un evento sobre una tabla (`INSERT`, `UPDATE`, `DELETE`) — a diferencia de una función o stored procedure, un trigger nunca se llama explícitamente, se dispara solo.

```sql
-- mantiene updated_at sincronizado sin que ninguna aplicación tenga que acordarse de hacerlo
CREATE FUNCTION set_updated_at() RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_orders_updated_at
  BEFORE UPDATE ON orders
  FOR EACH ROW
  EXECUTE FUNCTION set_updated_at();
```

### `BEFORE` vs `AFTER`

- **`BEFORE`**: corre antes de que el cambio se aplique — puede modificar los datos que se van a guardar (como el ejemplo de arriba, que pisa `NEW.updated_at`) o directamente cancelar la operación.
- **`AFTER`**: corre después de que el cambio ya se aplicó — típico para efectos secundarios que no deben alterar el dato en sí, como escribir a una tabla de auditoría.

```sql
-- AFTER: registra el cambio, sin modificar la fila que se acaba de insertar
CREATE TRIGGER trg_audit_orders
  AFTER INSERT ON orders
  FOR EACH ROW
  EXECUTE FUNCTION log_order_created();
```

### Trade-off

Un trigger garantiza que la lógica se ejecute **siempre**, sin importar desde qué servicio o script se hizo el cambio — ni un desarrollador con acceso directo a la DB puede saltearlo por accidente. El costo es el mismo problema de fondo que con stored procedures (ver [Trade-off: lógica en la DB vs en la aplicación](stored-procedures-vs-funciones.md#trade-off-lógica-en-la-db-vs-en-la-aplicación)), agravado: un trigger es lógica **invisible** desde el código de la aplicación — alguien leyendo el código de la app no tiene forma de saber que existe, hasta que lo descubre debuggeando un comportamiento inesperado. Por eso se usan con moderación, típicamente para invariantes de datos (auditoría, timestamps, validaciones de integridad) y no para lógica de negocio central.

### Por qué se ven cada vez menos en desarrollo de aplicaciones

En un backend de aplicación (web/mobile) moderno, triggers y stored procedures aparecen cada vez menos — la lógica que antes vivía en la DB hoy se escribe directo en el código de la aplicación usando un **ORM** (ver [ORM](../backend/controller-service-repository.md#orm-object-relational-mapping)), que ya resuelve buena parte de lo que un trigger resolvía (ej. `updated_at` automático vía un hook del ORM, no un trigger de SQL) sin la desventaja de ser invisible desde el código.

Donde sí siguen siendo comunes es en roles más orientados a **datos** (Data Engineer, Data Platform) — pipelines de ETL, integridad de un data warehouse, o sistemas legacy donde la lógica ya está ahí desde hace años y migrarla no es trivial. Vale la pena saber que existen y para qué sirven, pero no es lo que vas a escribir el día a día en un backend de aplicación típico con FastAPI/Django/Node.

---
Relacionado: [Stored procedures vs funciones](stored-procedures-vs-funciones.md), [ACID](acid-transacciones-isolation.md#acid) (los triggers son parte de qué garantiza la Consistency).
