#algo #tri #timsort #hybride

## Timsort

Algorithme hybride **Merge Sort + Insertion Sort** qui exploite les sous-séquences déjà triées (runs) dans les données réelles. Algorithme de tri par défaut de Python, Java et Android depuis 2002.

## Propriétés

- **Stable** : hérite de la stabilité de merge sort et insertion sort.
- O(n) best case (tableau trié ou presque).
- O(n log n) worst case garanti.
- O(n) espace auxiliaire.
- Conçu pour les **données réelles** qui contiennent souvent des runs naturels.

## Les deux idées clés

```
1. Runs naturels : les données réelles ont souvent des sous-séquences
   déjà triées (logs chronologiques, données partiellement mises à jour...).
   Timsort les détecte et les exploite.

2. Merge adaptatif : ne pas fusionner deux runs de tailles très déséquilibrées.
   Maintenir un stack de runs et fusionner selon des invariants de taille.
```

## Phase 1 — Détection et création des runs

```python
MIN_RUN = 32   # seuil empirique (entre 32 et 64 selon l'implémentation)

def find_run(arr, start, n):
    end = start + 1
    if end == n:
        return start, end

    if arr[end] < arr[start]:
        # Run décroissant → détecter et inverser
        while end < n and arr[end] < arr[end - 1]:
            end += 1
        arr[start:end] = arr[start:end][::-1]    # ✅ inverser pour le rendre croissant
    else:
        # Run croissant → étendre
        while end < n and arr[end] >= arr[end - 1]:
            end += 1

    # Si le run est trop court → compléter avec insertion sort
    if end - start < MIN_RUN:
        end = min(start + MIN_RUN, n)
        insertion_sort_range(arr, start, end)     # ✅ O(n) sur quasi-trié

    return start, end
```

## Phase 2 — Stack de runs et invariants de fusion

```python
# Timsort maintient un stack de runs (start, length)
# et respecte ces invariants pour équilibrer les fusions :
#
#   Soit A, B, C les 3 derniers runs du stack (C = sommet) :
#   Invariant 1 : len(A) > len(B) + len(C)
#   Invariant 2 : len(B) > len(C)
#
# Si un invariant est violé → fusionner B et C (ou A et B)

def timsort(arr):
    n       = len(arr)
    runs    = []
    pos     = 0

    while pos < n:
        start, end = find_run(arr, pos, n)
        runs.append((start, end - start))
        merge_collapse(arr, runs)            # ✅ vérifier et appliquer les invariants
        pos = end

    while len(runs) > 1:
        merge_at(arr, runs, -2)              # ✅ fusion finale des runs restants
```

## Phase 3 — Fusion avec Galloping

```python
# Galloping mode : si un run "gagne" k fois de suite,
# passer en recherche exponentielle pour avancer rapidement.
# Particulièrement efficace quand un run est beaucoup plus petit.

GALLOP_THRESHOLD = 7   # seuil pour activer le galloping

def merge_with_gallop(left, right):
    result      = []
    i = j       = 0
    left_wins   = 0
    right_wins  = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
            left_wins += 1; right_wins = 0
        else:
            result.append(right[j]); j += 1
            right_wins += 1; left_wins = 0

        # ✅ Activer le galloping si un côté domine
        if left_wins >= GALLOP_THRESHOLD or right_wins >= GALLOP_THRESHOLD:
            i, j = gallop(left, right, i, j, result)
            left_wins = right_wins = 0

    result.extend(left[i:]); result.extend(right[j:])
    return result
```

## Illustration — cas concret

```
Données réelles partiellement triées :
[1,3,5,7,|9,11,|2,4,6,8,10]
         run1   run2   run3

Timsort détecte 3 runs naturels.
Fusion : run1+run2 → [1,3,5,7,9,11]
         résultat + run3 → [1,2,3,4,5,6,7,8,9,10,11] ✅

vs Merge Sort classique qui diviserait le tableau indépendamment
des runs existants → travail inutile.
```

## Complexité

| Cas | Quand | Temps | Espace |
|-----|-------|-------|--------|
| Best | Tableau trié (1 run) | O(n) | O(1) |
| Near-sorted | Peu de runs | O(n log k) où k = nb runs | O(n) |
| Average | Données aléatoires | O(n log n) | O(n) |
| Worst | Aucun run naturel | O(n log n) | O(n) |

## Pourquoi Python utilise Timsort

```python
# Python sorted() = Timsort. Toujours stable, toujours O(n log n) worst.
arr = [5, 3, 1, 2, 4]
sorted(arr)                    # ✅ Timsort
arr.sort()                     # ✅ Timsort en place

# Sur des données quasi-triées (logs, timestamps, données mises à jour incrémentalement)
# Timsort est spectaculairement rapide — souvent O(n).
```

> [!tip] Lire les données avant de choisir l'algo
> Timsort illustre une philosophie : le meilleur algorithme dépend des **données réelles**, pas seulement de la complexité asymptotique. Les données ont presque toujours une structure partielle exploitable.

> [!info] Origine
> Timsort a été inventé par Tim Peters en 2002 pour Python. Il a été adopté par Java 7 (pour les objets), Android, et Swift. Il est formellement prouvé correct depuis 2015 (une implémentation Java avait un bug dans les invariants de merge, corrigé après preuve formelle).

> [!warning] Bug historique
> En 2015, des chercheurs ont formellement vérifié Timsort et trouvé un bug dans l'implémentation Java : les invariants du stack de runs n'étaient pas toujours respectés, causant un ArrayIndexOutOfBoundsException sur des tableaux de ~67M éléments. Corrigé dans Java 8u20.
