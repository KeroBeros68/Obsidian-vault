#algo #tri #selection-sort #classique

## Tri par sélection — Selection Sort

À chaque passe, **sélectionne le minimum** du sous-tableau non trié et le place à sa position finale. Minimise le nombre d'échanges.

## Propriétés

- **Non stable** dans la version classique (un swap peut enjamber des égaux).
- **En place** : O(1) espace auxiliaire.
- Toujours O(n²) comparaisons — pas d'optimisation possible pour les données déjà triées.
- **O(n) échanges** — optimal pour les situations où les échanges sont très coûteux.

## Implémentation

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i                          # ✅ supposer le min = position courante
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j                  # ✅ trouver le vrai minimum

        if min_idx != i:
            arr[i], arr[min_idx] = arr[min_idx], arr[i]  # ✅ un seul swap par passe

    return arr
```

## Version stable (insertion à la place du swap)

```python
def selection_sort_stable(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j

        # ✅ Décaler au lieu de swapper → préserve l'ordre relatif
        key = arr[min_idx]
        while min_idx > i:
            arr[min_idx] = arr[min_idx - 1]
            min_idx -= 1
        arr[i] = key

    return arr
```

## Illustration

```
Tableau : [64, 25, 12, 22, 11]

Passe 1 : min de [64,25,12,22,11] = 11 (idx=4)
  swap(0,4) → [11, 25, 12, 22, 64]  ← 11 en place ✅

Passe 2 : min de [25,12,22,64] = 12 (idx=2)
  swap(1,2) → [11, 12, 25, 22, 64]  ← 12 en place ✅

Passe 3 : min de [25,22,64] = 22 (idx=3)
  swap(2,3) → [11, 12, 22, 25, 64]  ← 22 en place ✅

Passe 4 : min de [25,64] = 25 (idx=3) → déjà en place
  → [11, 12, 22, 25, 64] ✅
```

## Complexité

| Cas | Temps | Échanges |
|-----|-------|---------|
| Best | O(n²) | O(1) si déjà trié |
| Average | O(n²) | O(n) |
| Worst | O(n²) | O(n) |

```
Nombre de comparaisons :
Passe i : n - i - 1 comparaisons
Total : Σ(i=0 à n-1) (n-i-1) = n(n-1)/2 = O(n²)   ← toujours, sans exception
```

## Comparaison avec Bubble Sort

| Critère | Selection Sort | Bubble Sort |
|---------|---------------|-------------|
| Comparaisons | O(n²) toujours | O(n²) / O(n) best |
| Échanges | **O(n)** ✅ | O(n²) |
| Stable | ❌ (version naïve) | ✅ |
| Cache | Meilleur (accès séquentiel) | Similaire |

> [!tip] Quand préférer Selection Sort
> Si les **échanges sont coûteux** (ex : éléments de grande taille en mémoire, ou I/O), selection sort est préférable à bubble sort : exactement n−1 échanges au maximum.

> [!warning] Toujours O(n²)
> Contrairement à bubble sort (O(n) avec flag) ou insertion sort (O(n) sur quasi-trié), selection sort ne peut pas détecter un tableau déjà trié. Il effectue toujours n(n-1)/2 comparaisons.
