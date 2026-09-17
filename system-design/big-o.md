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

<table>
<tr><th align="left">Estructura</th><th align="left">Access (avg)</th><th align="left">Search (avg)</th><th align="left">Insertion (avg)</th><th align="left">Deletion (avg)</th><th align="left">Access (worst)</th><th align="left">Search (worst)</th><th align="left">Insertion (worst)</th><th align="left">Deletion (worst)</th><th align="left">Space (worst)</th></tr>
<tr><td><strong>Array</strong></td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Stack</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Queue</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Singly-Linked List</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Doubly-Linked List</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Skip List</strong></td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td></tr>
<tr><td><strong>Hash Table</strong></td><td align="center" style="background:#e0e0e0;color:#000">N/A</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td><td align="center" style="background:#e0e0e0;color:#000">N/A</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Binary Search Tree</strong></td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Cartesian Tree</strong></td><td align="center" style="background:#e0e0e0;color:#000">N/A</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#e0e0e0;color:#000">N/A</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>B-Tree</strong></td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Red-Black Tree</strong></td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Splay Tree</strong></td><td align="center" style="background:#e0e0e0;color:#000">N/A</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#e0e0e0;color:#000">N/A</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>AVL Tree</strong></td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>KD Tree</strong></td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
</table>

## Tabla de referencia — algoritmos de sorting (arrays)

<table>
<tr><th align="left">Algoritmo</th><th align="left">Best</th><th align="left">Average</th><th align="left">Worst</th><th align="left">Space (worst)</th></tr>
<tr><td><strong>Quicksort</strong></td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#8bc34a;color:#000">O(log n)</td></tr>
<tr><td><strong>Mergesort</strong></td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Timsort</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Heapsort</strong></td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td></tr>
<tr><td><strong>Bubble Sort</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td></tr>
<tr><td><strong>Insertion Sort</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td></tr>
<tr><td><strong>Selection Sort</strong></td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td></tr>
<tr><td><strong>Tree Sort</strong></td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Shell Sort</strong></td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ef5350;color:#000">O(n (log n)²)</td><td align="center" style="background:#ef5350;color:#000">O(n (log n)²)</td><td align="center" style="background:#4caf50;color:#000">O(1)</td></tr>
<tr><td><strong>Bucket Sort</strong></td><td align="center" style="background:#4caf50;color:#000">O(n+k)</td><td align="center" style="background:#4caf50;color:#000">O(n+k)</td><td align="center" style="background:#ef5350;color:#000">O(n²)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
<tr><td><strong>Radix Sort</strong></td><td align="center" style="background:#66bb6a;color:#000">O(nk)</td><td align="center" style="background:#66bb6a;color:#000">O(nk)</td><td align="center" style="background:#66bb6a;color:#000">O(nk)</td><td align="center" style="background:#4caf50;color:#000">O(n+k)</td></tr>
<tr><td><strong>Counting Sort</strong></td><td align="center" style="background:#4caf50;color:#000">O(n+k)</td><td align="center" style="background:#4caf50;color:#000">O(n+k)</td><td align="center" style="background:#4caf50;color:#000">O(n+k)</td><td align="center" style="background:#4caf50;color:#000">O(k)</td></tr>
<tr><td><strong>Cubesort</strong></td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffa726;color:#000">O(n log n)</td><td align="center" style="background:#ffeb3b;color:#000">O(n)</td></tr>
</table>

---
Relacionado: [Algoritmos, Sorting y Estructuras de Datos en Python](../stacks/python/algoritmos-y-sorting.md) (aplicación concreta a `list`/`dict`/`set`/`heapq`/`bisect`), [Índices](../database/indices.md) (mismo espíritu de Big-O, a nivel de DB).
