# Big-O

<table width="100%"><tr><td align="center" bgcolor="#ffffff">
<img src="big-o-chart.png" width="600">
</td></tr></table>

Big-O mide **cómo crece** el tiempo (o la memoria) que necesita un algoritmo a medida que crece el tamaño del input — no cuántos milisegundos tarda exactamente. Dos algoritmos O(n) pueden tener tiempos reales muy distintos (uno con más overhead por operación que el otro), pero ambos van a duplicar su tiempo si el input se duplica; eso es lo que la notación captura, no el número absoluto.

## Cómo se deriva de código

Se cuenta la operación que domina a medida que `n` crece, y se descartan constantes y términos de menor orden — `O(2n + 100)` se escribe `O(n)`, porque para `n` grande el `100` y el `2` dejan de importar frente al crecimiento de `n`.

```python
def buscar(lista, objetivo):      # O(n) — en el peor caso recorre toda la lista
    for x in lista:
        if x == objetivo:
            return True
    return False

def buscar_anidado(lista):        # O(n²) — un loop adentro de otro, cada uno recorre n
    for i in lista:
        for j in lista:
            if i == j:
                ...
```

## Las clases de complejidad, de mejor a peor

| Notación | Nombre | Ejemplo típico |
|---|---|---|
| O(1) | Constante | Acceso a un índice de array, lookup en hash table |
| O(log n) | Logarítmica | Búsqueda binaria — cada paso descarta la mitad de lo que queda |
| O(n) | Lineal | Recorrer una lista una vez |
| O(n log n) | Linearítmica | Los algoritmos de sorting eficientes (Timsort, mergesort, quicksort) |
| O(n²) | Cuadrática | Loops anidados sobre la misma colección |
| O(2ⁿ) | Exponencial | Fuerza bruta probando todas las combinaciones posibles (ej. subsets) |

```python
def busqueda_binaria(lista_ordenada, objetivo):  # O(log n)
    inicio, fin = 0, len(lista_ordenada) - 1
    while inicio <= fin:
        medio = (inicio + fin) // 2
        if lista_ordenada[medio] == objetivo:
            return medio
        elif lista_ordenada[medio] < objetivo:
            inicio = medio + 1      # descarta la mitad izquierda
        else:
            fin = medio - 1         # descarta la mitad derecha
    return -1
```

Cada clase, en orden, crece **mucho** más rápido que la anterior — con `n = 1.000.000`, O(log n) son ~20 pasos, O(n) es un millón de pasos, y O(n²) es un billón. La diferencia entre elegir bien o mal la estructura/algoritmo no es un detalle menor a esa escala.

## Peor caso, caso promedio, mejor caso

Big-O casi siempre se habla en **peor caso** (worst case) por default, salvo que se aclare lo contrario — es la garantía más útil para diseñar un sistema, porque no depende de tener suerte con el input. Un algoritmo puede tener mejor caso O(1) (el elemento buscado es el primero) y peor caso O(n) (está al final, o no está) — reportar solo el mejor caso sería engañoso.

## Tiempo vs espacio

Big-O también mide **memoria**, no solo tiempo — un algoritmo puede ser más rápido a costa de usar más memoria (ej. guardar resultados ya calculados para no recalcularlos, *memoization*) o más lento pero con memoria constante. Es un trade-off explícito, no siempre se optimiza para lo mismo.

## Tabla de referencia — estructuras de datos

| Estructura | Access (avg) | Search (avg) | Insertion (avg) | Deletion (avg) | Access (worst) | Search (worst) | Insertion (worst) | Deletion (worst) | Space (worst) |
|---|---|---|---|---|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) | O(1) | O(n) | O(n) | O(n) | O(n) |
| Stack | O(n) | O(n) | O(1) | O(1) | O(n) | O(n) | O(1) | O(1) | O(n) |
| Queue | O(n) | O(n) | O(1) | O(1) | O(n) | O(n) | O(1) | O(1) | O(n) |
| Singly-Linked List | O(n) | O(n) | O(1) | O(1) | O(n) | O(n) | O(1) | O(1) | O(n) |
| Doubly-Linked List | O(n) | O(n) | O(1) | O(1) | O(n) | O(n) | O(1) | O(1) | O(n) |
| Skip List | O(log n) | O(log n) | O(log n) | O(log n) | O(n) | O(n) | O(n) | O(n) | O(n log n) |
| Hash Table | N/A | O(1) | O(1) | O(1) | N/A | O(n) | O(n) | O(n) | O(n) |
| Binary Search Tree | O(log n) | O(log n) | O(log n) | O(log n) | O(n) | O(n) | O(n) | O(n) | O(n) |
| Cartesian Tree | N/A | O(log n) | O(log n) | O(log n) | N/A | O(n) | O(n) | O(n) | O(n) |
| B-Tree | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Red-Black Tree | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Splay Tree | N/A | O(log n) | O(log n) | O(log n) | N/A | O(log n) | O(log n) | O(log n) | O(n) |
| AVL Tree | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| KD Tree | O(log n) | O(log n) | O(log n) | O(log n) | O(n) | O(n) | O(n) | O(n) | O(n) |

## Tabla de referencia — algoritmos de sorting (arrays)

| Algoritmo | Best | Average | Worst | Space (worst) |
|---|---|---|---|---|
| Quicksort | O(n log n) | O(n log n) | O(n²) | O(log n) |
| Mergesort | O(n log n) | O(n log n) | O(n log n) | O(n) |
| Timsort | O(n) | O(n log n) | O(n log n) | O(n) |
| Heapsort | O(n log n) | O(n log n) | O(n log n) | O(1) |
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) |
| Tree Sort | O(n log n) | O(n log n) | O(n²) | O(n) |
| Shell Sort | O(n log n) | O(n (log n)²) | O(n (log n)²) | O(1) |
| Bucket Sort | O(n+k) | O(n+k) | O(n²) | O(n) |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+k) |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) |
| Cubesort | O(n) | O(n log n) | O(n log n) | O(n) |

---
Relacionado: [Algoritmos, Sorting y Estructuras de Datos en Python](../stacks/python/algoritmos-y-sorting.md) (aplicación concreta a `list`/`dict`/`set`/`heapq`/`bisect`), [Índices](../database/indices.md) (mismo espíritu de Big-O, a nivel de DB).
