# Big-O

Big-O measures **how** the time (or memory) an algorithm needs **grows** as the input size grows — not how many milliseconds it takes exactly. Two O(n) algorithms can have very different real times (one with more overhead per operation than the other), but both will double their time if the input doubles; that's what the notation captures, not the absolute number.

## How it's derived from code

You count the operation that dominates as `n` grows, and drop constants and lower-order terms — `O(2n + 100)` is written `O(n)`, because for large `n` the `100` and the `2` stop mattering next to the growth of `n`.

```python
def search(items, target):        # O(n) — worst case scans the whole list
    for x in items:
        if x == target:
            return True
    return False

def nested_search(items):        # O(n²) — a loop inside another, each scanning n
    for i in items:
        for j in items:
            if i == j:
                ...
```

## Complexity classes, best to worst

| Notation | Name | Typical example |
|---|---|---|
| O(1) | Constant | Array index access, hash table lookup |
| O(log n) | Logarithmic | Binary search — each step discards half of what's left |
| O(n) | Linear | Scanning a list once |
| O(n log n) | Linearithmic | Efficient sorting algorithms (Timsort, mergesort, quicksort) |
| O(n²) | Quadratic | Nested loops over the same collection |
| O(2ⁿ) | Exponential | Brute force trying every possible combination (e.g. subsets) |

```python
def binary_search(sorted_list, target):  # O(log n)
    start, end = 0, len(sorted_list) - 1
    while start <= end:
        mid = (start + end) // 2
        if sorted_list[mid] == target:
            return mid
        elif sorted_list[mid] < target:
            start = mid + 1      # discards the left half
        else:
            end = mid - 1         # discards the right half
    return -1
```

```python
def merge_sort(items):    # O(n log n)
    if len(items) <= 1:
        return items
    mid = len(items) // 2
    left = merge_sort(items[:mid])   # log n levels of splitting in half
    right = merge_sort(items[mid:])
    return merge(left, right)        # each level does O(n) work merging

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):   # scans both halves once: O(n)
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    return result + left[i:] + right[j:]
```

`merge_sort` recursively splits the list in half (`log n` levels, like binary search) and at each level merges all the elements (`O(n)` work) — `log n` levels × `O(n)` per level = `O(n log n)` total. It's the same reason Timsort, mergesort, and quicksort share that complexity: divide and combine.

Each class, in order, grows **much** faster than the previous one — with `n = 1,000,000`, O(log n) is ~20 steps, O(n) is a million steps, and O(n²) is a trillion. The difference between choosing the right or wrong structure/algorithm isn't a minor detail at that scale.

## Worst case, average case, best case

Big-O is almost always discussed as **worst case** by default, unless stated otherwise — it's the most useful guarantee for designing a system, because it doesn't depend on getting lucky with the input. An algorithm can have a best case of O(1) (the item you're looking for is first) and a worst case of O(n) (it's at the end, or not there at all) — reporting only the best case would be misleading.

## Time vs space

Big-O also measures **memory**, not just time — an algorithm can be faster at the cost of using more memory (e.g. storing already-computed results to avoid recalculating them, *memoization*) or slower but with constant memory. It's an explicit trade-off, not always optimized for the same thing.

<table width="100%"><tr><td align="center" bgcolor="#ffffff">
<img src="big-o-chart.png" width="600">
</td></tr></table>

## Reference table — data structures

<table width="100%"><tr><td align="center" bgcolor="#ffffff">
<img src="big-o-data-structures.png" width="700">
</td></tr></table>

## Reference table — sorting algorithms (arrays)

<table width="100%"><tr><td align="center" bgcolor="#ffffff">
<img src="big-o-array-sorting.png" width="700">
</td></tr></table>

---
Related: [Algorithms, Sorting, and Data Structures in Python](../stacks/python/algoritmos-y-sorting.md) (concrete application to `list`/`dict`/`set`/`heapq`/`bisect`), [Indexes](../database/indexes.md) (the same Big-O spirit, at the DB level).
