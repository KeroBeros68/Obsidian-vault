#algo #tri #bucket-sort #linéaire

## Tri par paquets — Bucket Sort

Distribue les éléments dans des **seaux (buckets)** selon leur valeur, trie chaque seau indépendamment, puis concatène. Efficace quand les données sont **uniformément distribuées**.

## Propriétés

- **O(n + k)** en moyenne avec distribution uniforme (k = nombre de seaux).
- **Stable** si le tri interne de chaque seau est stable.
- Nécessite de connaître a priori la plage des valeurs.
- O(n²) worst case si tous les éléments tombent dans le même seau.

## Implémentation — flottants dans [0, 1)

```python
def bucket_sort(arr, n_buckets=None):
    if not arr:
        return arr

    n         = len(arr)
    n_buckets = n_buckets or n
    min_val   = min(arr)
    max_val   = max(arr)
    rng       = max_val - min_val

    # Phase 1 : distribuer dans les seaux — O(n)
    buckets = [[] for _ in range(n_buckets)]
    for x in arr:
        if rng == 0:
            idx = 0
        else:
            idx = int((x - min_val) / rng * (n_buckets - 1))  # ✅ normaliser
        buckets[idx].append(x)

    # Phase 2 : trier chaque seau — O(k * m log m) où m = taille moyenne
    for bucket in buckets:
        bucket.sort()                        # ✅ insertion sort ou autre sur petit seau

    # Phase 3 : concaténer — O(n)
    return [x for bucket in buckets for x in bucket]
```

## Bucket sort pour entiers avec comptage interne

```python
def bucket_sort_int(arr, n_buckets=10):
    if not arr:
        return arr

    min_val, max_val = min(arr), max(arr)
    if min_val == max_val:
        return arr

    size    = (max_val - min_val) / n_buckets
    buckets = [[] for _ in range(n_buckets)]

    for x in arr:
        idx = min(int((x - min_val) / size), n_buckets - 1)  # ✅ clamp au dernier seau
        buckets[idx].append(x)

    result = []
    for bucket in buckets:
        result.extend(sorted(bucket))        # ✅ insertion sort implicite via sorted()
    return result
```

## Illustration

```
Tableau : [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12, 0.23, 0.68]
n_buckets = 5

Distribution :
  Seau 0 [0.0–0.2) : [0.17, 0.12]
  Seau 1 [0.2–0.4) : [0.39, 0.26, 0.21, 0.23]
  Seau 2 [0.4–0.6) : []
  Seau 3 [0.6–0.8) : [0.78, 0.72, 0.68]
  Seau 4 [0.8–1.0] : [0.94]

Tri interne :
  Seau 0 : [0.12, 0.17]
  Seau 1 : [0.21, 0.23, 0.26, 0.39]
  Seau 3 : [0.68, 0.72, 0.78]

Concaténation : [0.12, 0.17, 0.21, 0.23, 0.26, 0.39, 0.68, 0.72, 0.78, 0.94] ✅
```

## Complexité

| Cas | Quand | Temps |
|-----|-------|-------|
| Best | Distribution uniforme, n seaux | O(n) |
| Average | Distribution uniforme | O(n + k) |
| Worst | Tous dans le même seau | O(n²) |

```
Average avec n seaux et distribution uniforme :
- Chaque seau contient en moyenne 1 élément
- Tri de chaque seau : O(1)
- Total : O(n) ✅

Si la distribution est uniforme et n_buckets = n → O(n) en espérance.
```

## Variante : Bucket sort pour chaînes (par longueur)

```python
def bucket_sort_strings(arr):
    if not arr:
        return arr
    max_len = max(len(s) for s in arr)
    buckets = [[] for _ in range(max_len + 1)]
    for s in arr:
        buckets[len(s)].append(s)
    return [s for b in buckets for s in sorted(b)]
```

## Choisir le nombre de seaux

```
Trop peu de seaux → seaux surchargés → O(n²) si un seau domine
Trop de seaux     → overhead d'allocation + cache misses

Règle empirique : n_buckets ≈ √n ou n_buckets = n pour O(n) théorique.
```

> [!tip] Idéal pour les flottants uniformes
> Bucket sort est le tri naturel pour les nombres flottants dans [0, 1) générés aléatoirement. En espérance O(n), surpassant tous les tris par comparaisons.

> [!warning] Sensible à la distribution
> Sur une distribution non uniforme (ex : loi de Pareto, données groupées), bucket sort dégénère. Analyser la distribution des données avant de l'utiliser.

> [!info] Lien avec Counting et Radix
> Counting sort = bucket sort avec un seau par valeur entière.
> Radix sort = bucket sort appliqué chiffre par chiffre avec b seaux.
> Tous trois brisent la borne Ω(n log n) en n'utilisant pas de comparaisons directes entre éléments.
