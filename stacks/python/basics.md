# Python — Basics

## Python 2 vs Python 3

Python 2 lost official support in 2020 — today any new project is Python 3, but it's worth knowing the language's history to understand where certain conventions come from:

- `print` is a function in Python 3 (`print("hello")`); it was a statement in Python 2 (`print "hello"`).
- Division: `5 / 2` gives `2.5` in Python 3 (float by default); in Python 2 it gave `2` (integer division if both are int). Explicit integer division in Python 3: `5 // 2`.
- Strings: in Python 3 all strings are Unicode by default; in Python 2 you had to mark them by hand (`u"text"`) or you'd end up with encoding bugs.
- `range()` in Python 3 returns a lazy object (generator-like); in Python 2 it returned a full list in memory (`xrange()` was the lazy version).

## `pip` — the package installer

`pip` installs packages from PyPI (Python Package Index). `pip3` is the same, but explicit that it points to Python 3's `pip` (relevant on systems where Python 2 and 3 coexist — increasingly uncommon).

```bash
pip install fastapi
pip install fastapi==0.110.0   # specific version
pip uninstall fastapi
pip list                        # packages installed in the active environment
```

## `requirements.txt`

A plain text file listing the project's dependencies (and optionally their exact versions), so anyone can reproduce the same environment.

```
fastapi==0.110.0
uvicorn==0.29.0
sqlalchemy>=2.0,<3.0
```

```bash
pip freeze > requirements.txt     # generates the file with the exact versions currently installed
pip install -r requirements.txt   # installs everything the file lists
```

`pip freeze` dumps **everything** installed in the environment, including transitive dependencies (the ones another library installed, not something requested directly) — that's why a `requirements.txt` generated this way can have 50 lines even if the project only declares 5 direct dependencies. More modern tools (`uv`, `poetry`) separate the declared dependencies from the full lockfile.

## `if __name__ == "__main__"`

Every Python module has a `__name__` variable. If the file is run directly (`python main.py`), `__name__` equals `"__main__"`. If the file is imported from another one (`import main`), `__name__` equals the module's name (`"main"`), not `"__main__"`.

```python
def main():
    print("Running the app")

if __name__ == "__main__":
    main()   # only runs if the file is executed directly, not if it's imported
```

**Why it matters**: without this check, any "startup" code in the file would also run every time someone else imports it — even if they only wanted to reuse a function defined there, without running the whole app.

## Module vs package

A **module** is a single `.py` file. A **package** is a folder with several modules inside, marked as importable with an `__init__.py` (in modern Python that file can be empty or not exist at all — *namespace packages* — but it's still the most common convention to find one).

```
my_package/
├── __init__.py
├── models.py      # one module
└── utils.py        # another module
```

```python
from my_package import models   # imports the module from the package
```

---
Related: [General Syntax](syntax.md).
