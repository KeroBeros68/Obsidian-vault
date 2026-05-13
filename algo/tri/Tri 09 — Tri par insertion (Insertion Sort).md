#algo #tri #insertion-sort #classique

## Tri par insertion — Insertion Sort

Maintient une **partie gauche triée** et insère chaque nouvel élément à sa bonne position, en décalant les éléments plus grands vers la droite. Analogue au tri d'une main de cartes.

## Propriétés

- **Stable** : les éléments égaux ne sont jamais échangés.
- **En place** : O(1) espace auxiliaire.
- **O(n) best case** : idéal sur les tableaux **presque triés** (peu d'inversions).
- Meilleur que bubble et selection pour les petits tableaux (n ≤ 20–50).
- Utilisé comme sous-routine dans Timsort et Introsort.

## Implémentation

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]                         # ✅ élément à insérer
        j   = i - 1

        while j >= 0 and arr[j] > key:       # ✅ décaler les plus grands vers la droite
            arr[j + 1] = arr[j]
            j -= 1

        arr[j + 1] = key                     # ✅ insérer à la bonne position
    return arr
```

## Variante avec recherche binaire (Binary Insertion Sort)

```python
import bisect

def binary_insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        # ✅ bisect_left : O(log i) pour trouver la position
        pos = bisect.bisect_left(arr, key, 0, i)
        # ⚠️ le décalage reste O(i) — améliore les comparaisons, pas les mouvements
        arr[pos+1:i+1] = arr[pos:i]
        arr[pos] = key
    return arr
```

## Illustration

```
Tableau : [5, 2, 4, 6, 1, 3]

i=1 : key=2, [5]>2 → décaler → [_,5,4,6,1,3] → insérer → [2,5,4,6,1,3]
i=2 : key=4, [5]>4 → décaler → [2,_,5,6,1,3] → insérer → [2,4,5,6,1,3]
i=3 : key=6, [5]≤6 → stop    →                             [2,4,5,6,1,3]
i=4 : key=1, décaler [6,5,4,2] → [_,2,4,5,6,3] → insérer → [1,2,4,5,6,3]
i=5 : key=3, décaler [5,4]     → [1,2,_,4,5,6] → insérer → [1,2,3,4,5,6] ✅

Nombre de décalages = nombre d'inversions dans le tableau original.
```

## Notion d'inversions

```
Une inversion = paire (i,j) avec i<j et arr[i]>arr[j].

Tableau trié    : 0 inversion   → O(n)   ✅ best case
Tableau aléat.  : n(n-1)/4 inversions → O(n²) average
Tableau inversé : n(n-1)/2 inversions → O(n²) worst case

Insertion sort fait exactement autant de décalages qu'il y a d'inversions.
```

## Complexité

| Cas | Quand | Temps | Décalages |
|-----|-------|-------|----------|
| Best | Tableau trié (0 inversions) | O(n) | 0 |
| Average | Permutation aléatoire | O(n²) | O(n²) |
| Worst | Tableau trié à l'envers | O(n²) | O(n²) |

## Sous-routine dans les tris hybrides

```python
# Timsort et Introsort utilisent insertion sort pour les petits sous-tableaux
THRESHOLD = 16   # typiquement 16-32 éléments

def introsort_hybrid(arr, low, high, depth_limit):
    if high - low < THRESHOLD:
        insertion_sort_range(arr, low, high)  # ✅ optimal sur petits tableaux
        return
    # ... suite quicksort / heapsort

def insertion_sort_range(arr, low, high):
    for i in range(low + 1, high + 1):
        key = arr[i]
        j   = i - 1
        while j >= low and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key
```

> [!tip] Meilleur choix pour les petits tableaux
> En dessous de ~20 éléments, les constantes cachées d'insertion sort (simple, peu de mémoire) le rendent plus rapide que merge sort ou quick sort. C'est pourquoi Timsort et Introsort y recourent.

> [!tip] Analogue aux cartes à jouer
> Trier une main de cartes : on prend chaque carte et on la glisse à sa place dans la main déjà triée. C'est exactement l'insertion sort.

> [!info] Binary insertion sort
> La variante binaire réduit le nombre de **comparaisons** à O(n log n) total, mais le nombre de **décalages** reste O(n²). Utile si la comparaison est coûteuse mais les déplacements mémoire sont rapides.
