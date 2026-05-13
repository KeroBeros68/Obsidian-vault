#algo #tri #quicksort #diviser-pour-régner

## Tri rapide — Quick Sort

Choisit un **pivot**, partitionne le tableau en éléments ≤ pivot et éléments > pivot, puis trie récursivement chaque partie. Le tri par comparaisons le plus rapide **en pratique**.

## Propriétés

- **O(n log n) en moyenne**, mais O(n²) dans le pire cas (pivot mal choisi).
- **Non stable** : les éléments égaux peuvent changer d'ordre.
- **En place** : O(log n) espace (pile de récursion), O(1) hors récursion.
- Les constantes cachées sont petites → plus rapide que merge sort en pratique sur les tableaux.

## Implémentation — partition de Lomuto

```python
def quicksort(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    if low < high:
        p = partition(arr, low, high)
        quicksort(arr, low, p - 1)          # ✅ partie gauche
        quicksort(arr, p + 1, high)         # ✅ partie droite

def partition(arr, low, high):
    pivot = arr[high]                       # ⚠️ pivot = dernier élément (naïf)
    i = low - 1                             # frontière des éléments ≤ pivot

    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]

    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1                            # ✅ position finale du pivot
```

## Partition de Hoare (plus efficace)

```python
def partition_hoare(arr, low, high):
    pivot = arr[(low + high) // 2]          # ✅ pivot au centre
    i, j  = low - 1, high + 1

    while True:
        i += 1
        while arr[i] < pivot:
            i += 1
        j -= 1
        while arr[j] > pivot:
            j -= 1
        if i >= j:
            return j
        arr[i], arr[j] = arr[j], arr[i]
```

## Choix du pivot — stratégies

```python
import random

# ❌ Toujours premier ou dernier → O(n²) sur tableau trié
pivot = arr[low]

# ✅ Aléatoire → O(n log n) en espérance, élimine les entrées adversariales
idx   = random.randint(low, high)
arr[idx], arr[high] = arr[high], arr[idx]
pivot = arr[high]

# ✅ Médiane de trois → bon compromis déterministe
mid   = (low + high) // 2
candidates = sorted([(arr[low], low), (arr[mid], mid), (arr[high], high)])
_, pivot_idx = candidates[1]
arr[pivot_idx], arr[high] = arr[high], arr[pivot_idx]
pivot = arr[high]
```

## Introsort — la version production

```python
# Introsort = Quick Sort + Heap Sort + Insertion Sort
# Utilisé par C++ std::sort, .NET Array.Sort

def introsort(arr, low, high, depth_limit):
    size = high - low + 1
    if size <= 16:
        insertion_sort(arr, low, high)      # ✅ optimal sur petits tableaux
        return
    if depth_limit == 0:
        heap_sort(arr, low, high)           # ✅ O(n log n) garanti si récursion trop profonde
        return
    p = partition(arr, low, high)
    introsort(arr, low, p - 1, depth_limit - 1)
    introsort(arr, p + 1, high, depth_limit - 1)

# depth_limit initial = 2 * floor(log2(n))
```

## Illustration — déroulement

```
[3, 6, 8, 10, 1, 2, 1]   pivot = 1 (dernier)

Partition :
i=-1, j parcourt [3,6,8,10,1,2]
  j=4 : arr[4]=1 ≤ 1 → i=0, swap(arr[0],arr[4]) → [1,6,8,10,3,2,1]
  j=5 : arr[5]=2 > 1  → pas de swap
Swap final : arr[1],arr[6] → [1,1,8,10,3,2,6]
pivot en position 1

Récursion gauche  : [1]         → trié ✅
Récursion droite  : [8,10,3,2,6] → ...
```

## Complexité

| Cas | Quand | Comparaisons | Récursion |
|-----|-------|-------------|-----------|
| Best | Pivot = médiane exacte | O(n log n) | O(log n) |
| Average | Pivot aléatoire | O(n log n) | O(log n) |
| Worst | Pivot = min ou max à chaque fois | O(n²) | O(n) ⚠️ |

```
Récurrence moyenne : T(n) = 2·T(n/2) + O(n) → O(n log n)
Récurrence worst   : T(n) = T(n-1) + O(n)   → O(n²)
```

> [!warning] Worst case O(n²)
> Sur un tableau déjà trié avec pivot = dernier élément, le pivot est toujours le minimum. Chaque partition ne réduit la taille que de 1. → Toujours utiliser un pivot aléatoire ou médiane de trois.

> [!tip] Plus rapide que merge sort en pratique
> Quick sort a une meilleure localité de cache (accès séquentiels) et moins d'allocations mémoire que merge sort. Les constantes cachées dans le O(n log n) sont inférieures.

> [!info] Tail call optimization
> Après la partition, trier d'abord la **petite** moitié et optimiser la récursion terminale sur la grande. Réduit la pile de récursion de O(n) à O(log n) dans le pire cas.
