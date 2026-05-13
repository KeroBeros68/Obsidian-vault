#algo #tri #bubble-sort #classique

## Tri à bulles — Bubble Sort

Compare répétitivement des **paires adjacentes** et les échange si elles sont dans le mauvais ordre. Les grandes valeurs "remontent" vers la fin comme des bulles.

## Propriétés

- **Stable** : deux éléments égaux ne sont jamais échangés.
- **En place** : O(1) espace auxiliaire.
- Pédagogique mais rarement utilisé en production — O(n²) en moyenne et worst case.
- **O(n) best case** avec l'optimisation "flag" : détecte si le tableau est déjà trié.

## Implémentation naïve

```python
def bubble_sort_naive(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):       # ✅ n-i-1 : les i derniers sont déjà en place
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr
```

## Implémentation optimisée — flag de détection

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False                      # ✅ flag : détecte l'absence d'échange
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:
            break                            # ✅ aucun échange → tableau trié → O(n)
    return arr
```

## Illustration — une passe complète

```
Tableau : [5, 3, 8, 1, 2]

Passe 1 :
  [5,3] → swap → [3,5,8,1,2]
  [5,8] → ok   → [3,5,8,1,2]
  [8,1] → swap → [3,5,1,8,2]
  [8,2] → swap → [3,5,1,2,8]  ← 8 en place ✅

Passe 2 :
  [3,5] → ok
  [5,1] → swap → [3,1,5,2,8]
  [5,2] → swap → [3,1,2,5,8]  ← 5 en place ✅

...

Final : [1,2,3,5,8] ✅
```

## Complexité

| Cas | Quand | Temps | Échanges |
|-----|-------|-------|---------|
| Best | Tableau trié (avec flag) | O(n) | 0 |
| Average | Permutation aléatoire | O(n²) | O(n²) |
| Worst | Tableau trié à l'envers | O(n²) | O(n²) |

## Variante : Bubble sort avec frontière dynamique

```python
def bubble_sort_optimized(arr):
    n = len(arr)
    while n > 1:
        last_swap = 0
        for j in range(n - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                last_swap = j + 1            # ✅ dernière position échangée
        n = last_swap                        # ✅ tout après last_swap est trié
    return arr
```

> [!tip] Utilité pédagogique
> Bubble sort illustre parfaitement la notion de **invariant de boucle** : après la k-ième passe, les k plus grands éléments sont à leur position finale.

> [!warning] Jamais en production
> O(n²) comparaisons ET O(n²) échanges (coûteux si les échanges sont lents). Même l'insertion sort est meilleur : O(n²) comparaisons mais O(n) échanges en moyenne.
