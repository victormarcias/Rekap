# SQL Injection

Insertar código SQL a través de un input que se concatena directo en una query, en vez de tratarse como un dato — el motor de base de datos no tiene forma de distinguir "esto es parte de la instrucción" de "esto es un valor" si ambas cosas llegan mezcladas en el mismo string.

```python
# ❌ vulnerable: el input del usuario se pega directo en el SQL
usuario_id = request.args["id"]
query = f"SELECT * FROM users WHERE id = {usuario_id}"

# si usuario_id = "1 OR 1=1"        → la condición siempre es verdadera, devuelve TODOS los usuarios
# si usuario_id = "1; DROP TABLE users; --"  → borra la tabla entera (si el driver permite multi-statement)

# ✅ seguro: parametrized query — el driver manda el SQL y el dato por separado,
# nunca se concatenan como texto
cursor.execute("SELECT * FROM users WHERE id = %s", (usuario_id,))
```

## El error de pensar que un ORM te protege solo

Un ORM parametriza automáticamente cuando usás su API — pero si armás el SQL a mano dentro del mismo proyecto (con f-strings, `.format()`, o concatenación), la vulnerabilidad sigue ahí, ORM o no:

```python
# ❌ sigue siendo vulnerable — el ORM no interviene, es un string armado a mano
session.execute(f"SELECT * FROM users WHERE email = '{email}'")

# ✅ el ORM parametriza porque estás usando su API, no un string propio
session.query(User).filter(User.email == email)
```

## Cómo se previene

- **Parametrized queries / prepared statements**: siempre — nunca interpolar un valor del usuario directo en el SQL, ni siquiera "solo para un caso puntual".
- **ORM usado como corresponde**: evita el problema por diseño, siempre que no se rompa la abstracción con SQL crudo armado a mano.
- **Mínimo privilegio en la DB**: el usuario de conexión de la aplicación no debería tener permiso de `DROP`/`ALTER` si la app nunca lo necesita en producción — limita el daño incluso si algo se cuela.

---
Relacionado: [Motores de SQL](../database/motores-de-sql.md), [Controller / Service / Repository](../backend/controller-service-repository.md#orm-object-relational-mapping), [SSRF](ssrf.md).
