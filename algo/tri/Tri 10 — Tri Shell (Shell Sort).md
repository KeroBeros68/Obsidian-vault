#algo #tri #shell-sort #classique

## Tri Shell — Shell Sort

Généralisation de l'insertion sort : trie d'abord les éléments **distants d'un gap h**, puis réduit h progressivement jusqu'à 1. Casse les grandes inversions tôt pour accélérer la passe finale.

## Propriétés

- **Non stable** : les swaps à grand gap peuvent inverser l'ordre relatif des égaux.
- **En place** : O(1) espace auxiliaire.
- Complexité dépend de la **séquence de gaps** choisie — pas de consensus sur la séquence optimale.
- Plus rapide qu'insertion sort sur les grands tableaux, mais dépassé par merge/quick sort.

## Implémentation

```python
def shell_sort(arr, gaps=None):
    n = len(arr)
    if gaps is None:
        # Séquence de Knuth : 1, 4, 13, 40, 121, ...  (h = 3h + 1)
        gap = 1
        while gap < n // 3:
            gap = 3 * gap + 1
        gaps = []
        while gap >= 1:
            gaps.append(gap)
            gap //= 3

    for gap in gaps:                         # ✅ décroître les gaps
        for i in range(gap, n):
            key = arr[i]
            j   = i
            while j >= gap and arr[j - gap] > key:   # ✅ insertion sort à distance gap
                arr[j] = arr[j - gap]
                j -= gap
            arr[j] = key

    return arr
```

## Séquences de gaps et leurs complexités

```python
# Shell (1959) : n/2, n/4, ..., 1
gaps_shell = [n // (2**k) for k in range(1, ...)]   # O(n²) worst

# Knuth (1973) : 1, 4, 13, 40, 121, ...
gaps_knuth = [1, 4, 13, 40, 121, 364, ...]           # O(n^1.5)

# Hibbard (1963) : 1, 3, 7, 15, 31, ... (2^k - 1)
gaps_hibbard = [2**k - 1 for k in range(1, ...)]     # O(n^1.5)

# Ciura (2001) : séquence empiriquement optimale
gaps_ciura = [701, 301, 132, 57, 23, 10, 4, 1]       # ≈ O(n^1.25) en pratique ✅
```

## Illustration — gap=4 puis gap=1

```
Tableau : [9, 8, 3, 7, 5, 6, 4, 1]  n=8

Gap = 4 : trier les paires (0,4), (1,5), (2,6), (3,7)
  (9,5) → swap → [5, 8, 3, 7, 9, 6, 4, 1]
  (8,6) → swap → [5, 6, 3, 7, 9, 8, 4, 1]
  (3,4) → ok   → [5, 6, 3, 7, 9, 8, 4, 1]
  (7,1) → swap → [5, 6, 3, 1, 9, 8, 4, 7]
Après gap=4 : [5, 6, 3, 1, 9, 8, 4, 7]

Gap = 1 : insertion sort classique
Les grandes inversions ont été corrigées → peu de décalages restants.
Final : [1, 3, 4, 5, 6, 7, 8, 9] ✅
```

## Complexité

| Séquence | Worst case | Average |
|----------|-----------|---------|
| Shell (n/2) | O(n²) | O(n²) |
| Hibbard | O(n^1.5) | O(n^1.25) |
| Knuth | O(n^1.5) | — |
| Ciura | inconnu | ≈ O(n^1.25) |

> [!info] Complexité inconnue
> La complexité exacte de Shell sort avec la séquence de Ciura n'est pas démontrée formellement. C'est l'un des rares algorithmes classiques avec une complexité encore ouverte.

> [!tip] Cas d'usage pratique
> Shell sort est utile dans les **environnements embarqués** (peu de mémoire, pas de récursion) où merge sort et quick sort sont impraticables. Compact, en place, et nettement meilleur que O(n²) pur.

> [!warning] Sensibilité aux gaps
> La séquence de gaps est critique. Avec la séquence naïve de Shell (n/2), le worst case reste O(n²). Toujours préférer Knuth ou Ciura.
