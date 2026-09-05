# Python — Básico

## Python 2 vs Python 3

Python 2 dejó de tener soporte oficial en 2020 — hoy cualquier proyecto nuevo es Python 3, pero vale la pena conocer la historia del lenguaje para entender de dónde vienen ciertas convenciones:

- `print` es una función en Python 3 (`print("hola")`); era una sentencia en Python 2 (`print "hola"`).
- División: `5 / 2` da `2.5` en Python 3 (float por default); en Python 2 daba `2` (división entera si ambos son int). División entera explícita en Python 3: `5 // 2`.
- Strings: en Python 3 todos los strings son Unicode por default; en Python 2 había que marcarlos a mano (`u"texto"`) o terminabas con bugs de encoding.
- `range()` en Python 3 devuelve un objeto perezoso (tipo generator); en Python 2 devolvía una lista completa en memoria (`xrange()` era la versión perezosa).

## `pip` — el instalador de paquetes

`pip` instala paquetes desde PyPI (Python Package Index). `pip3` es lo mismo, pero explícito de que apunta al `pip` de Python 3 (relevante en sistemas donde conviven Python 2 y 3 — cada vez menos común).

```bash
pip install fastapi
pip install fastapi==0.110.0   # versión específica
pip uninstall fastapi
pip list                        # paquetes instalados en el entorno activo
```

## `requirements.txt`

Archivo de texto plano que lista las dependencias del proyecto (y opcionalmente sus versiones exactas), para que cualquiera pueda reproducir el mismo entorno.

```
fastapi==0.110.0
uvicorn==0.29.0
sqlalchemy>=2.0,<3.0
```

```bash
pip freeze > requirements.txt     # genera el archivo con las versiones exactas instaladas ahora
pip install -r requirements.txt   # instala todo lo que lista el archivo
```

`pip freeze` vuelca **todo** lo instalado en el entorno, incluyendo dependencias transitivas (las que instaló otra librería, no algo pedido directamente) — por eso un `requirements.txt` generado así puede tener 50 líneas aunque el proyecto solo declare 5 dependencias directas. Herramientas más modernas (`uv`, `poetry`) separan las dependencias declaradas del lockfile completo.

## `if __name__ == "__main__"`

Cada módulo de Python tiene una variable `__name__`. Si el archivo se corre directamente (`python main.py`), `__name__` vale `"__main__"`. Si el archivo se importa desde otro (`import main`), `__name__` vale el nombre del módulo (`"main"`), no `"__main__"`.

```python
def main():
    print("Corriendo la app")

if __name__ == "__main__":
    main()   # solo corre si el archivo se ejecuta directo, no si se importa
```

**Por qué importa**: sin este chequeo, cualquier código "de arranque" en el archivo se ejecutaría también cada vez que alguien más lo importa — incluso si solo quería reusar una función definida ahí, sin correr la app entera.

## Módulo vs paquete

Un **módulo** es un solo archivo `.py`. Un **paquete** es una carpeta con varios módulos adentro, marcada como importable con un `__init__.py` (en Python moderno ese archivo puede estar vacío o directamente no existir — *namespace packages* — pero sigue siendo la convención más común encontrarlo).

```
mi_paquete/
├── __init__.py
├── models.py      # un módulo
└── utils.py        # otro módulo
```

```python
from mi_paquete import models   # importa el módulo desde el paquete
```

---
Relacionado: [Sintaxis general](sintaxis.md).
