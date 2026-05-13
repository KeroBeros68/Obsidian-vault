#algo #tri #heapsort #tas

## Tri par tas — Heap Sort

Construit un **tas max** à partir du tableau, puis extrait le maximum répétitivement pour remplir le tableau de droite à gauche. Tri **en place** avec O(n log n) garanti.

## Propriétés

- O(n log n) dans **tous les cas** — pas de dégradation worst case.
- **En place** : O(1) espace auxiliaire.
- **Non stable** : les éléments égaux peuvent changer d'ordre.
- Plus lent que quick sort en pratique (mauvaise localité de cache).
- Utilisé dans introsort comme filet de sécurité quand quick sort dégénère.

## Rappel : propriété de tas max

```
Dans un tas max représenté en tableau :
- Parent de i      : (i - 1) // 2
- Enfant gauche    : 2*i + 1
- Enfant droit     : 2*i + 2
- Invariant        : arr[parent] >= arr[enfant]  ← toujours

Index :  0   1   2   3   4   5   6
Valeur : 9   7   8   3   4   5   2

        9
       / \
      7   8
     / \ / \
    3  4 5  2   ← tas max valide ✅
```

## Implémentation

```python
def heap_sort(arr):
    n = len(arr)

    # Phase 1 : construire le tas max — O(n)
    for i in range(n // 2 - 1, -1, -1):    # ✅ partir des feuilles vers la racine
        heapify(arr, n, i)

    # Phase 2 : extraire le max répétitivement — O(n log n)
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]     # ✅ max en position finale
        heapify(arr, i, 0)                  # ✅ restaurer le tas sur n-1 éléments

def heapify(arr, n, i):
    largest = i
    left    = 2 * i + 1
    right   = 2 * i + 2

    if left < n and arr[left] > arr[largest]:
        largest = left
    if right < n and arr[right] > arr[largest]:
        largest = right

    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)            # ✅ propager vers le bas (sift-down)
```

## Illustration — les deux phases

```
Tableau initial : [4, 10, 3, 5, 1]

Phase 1 — Build heap O(n) :
Partir de i = n//2-1 = 1 :
  heapify(i=1) : arr[1]=10 > arr[3]=5, arr[4]=1 → ok
  heapify(i=0) : arr[0]=4 < arr[1]=10 → swap → [10, 4, 3, 5, 1]
                 heapify(i=1) : arr[1]=4 < arr[3]=5 → swap → [10, 5, 3, 4, 1]
Tas max : [10, 5, 3, 4, 1] ✅

Phase 2 — Extract max O(n log n) :
swap(0,4) → [1, 5, 3, 4, | 10]  heapify(n=4) → [5, 4, 3, 1, | 10]
swap(0,3) → [1, 4, 3, | 5, 10]  heapify(n=3) → [4, 1, 3, | 5, 10]
swap(0,2) → [3, 1, | 4, 5, 10]  heapify(n=2) → [3, 1, | 4, 5, 10]
swap(0,1) → [1, | 3, 4, 5, 10]
Résultat  : [1, 3, 4, 5, 10] ✅
```

## Pourquoi la construction du tas est O(n) et non O(n log n)

```
Intuitif : les nœuds proches des feuilles font peu de travail.
Formellement :
  - n/2 nœuds au niveau 0 (feuilles)  → 0 opérations chacun
  - n/4 nœuds au niveau 1             → 1 opération chacun
  - n/8 nœuds au niveau 2             → 2 opérations chacun
  - ...
  Σ (n / 2^(h+1)) * h = O(n)   (série géométrique) ✅
```

## Complexité

| Cas | Temps | Espace |
|-----|-------|--------|
| Best | O(n log n) | O(1) |
| Average | O(n log n) | O(1) |
| Worst | O(n log n) | O(1) |

> [!tip] O(1) espace — le seul parmi les O(n log n) garantis
> Ni le tri fusion (O(n)), ni le tri rapide (O(log n) pile) n'atteignent O(1). Le tri par tas est le seul à combiner O(n log n) garanti ET O(1) espace.

> [!warning] Non stable et cache-unfriendly
> Le tri par tas est non stable et accède au tableau de façon non séquentielle (sauts entre parent et enfants). Cela crée beaucoup de cache misses, le rendant 2–5× plus lent que quick sort en pratique malgré la même complexité asymptotique.

> [!info] Lien avec le tas de Fibonacci
> Le tas max ici est un **tas binaire** (arbre binaire complet). Le tas de Fibonacci ([[Graphes 09 — Tas de Fibonacci]]) est une structure plus sophistiquée optimisant decrease-key pour Dijkstra/Prim, mais inadaptée au tri.
