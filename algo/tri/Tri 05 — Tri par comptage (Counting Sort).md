#algo #tri #counting-sort #linéaire

## Tri par comptage — Counting Sort

Compte les occurrences de chaque valeur, puis reconstruit le tableau trié **sans comparer les éléments entre eux**. Brise la borne Ω(n log n) des tris par comparaisons.

## Propriétés

- **O(n + k)** où k = étendue des valeurs (max − min + 1).
- **Stable** : l'ordre relatif des éléments égaux est préservé (avec l'implémentation cumulative).
- Applicable uniquement sur des **entiers** (ou valeurs discrètes mappables à des entiers).
- Inefficace si k >> n (beaucoup de valeurs possibles mais peu présentes).

## Implémentation — version stable

```python
def counting_sort(arr, key=lambda x: x):
    if not arr:
        return arr

    min_val = min(key(x) for x in arr)
    max_val = max(key(x) for x in arr)
    k       = max_val - min_val + 1

    # Phase 1 : compter les occurrences — O(n)
    count = [0] * k
    for x in arr:
        count[key(x) - min_val] += 1       # ✅ décalage par min_val

    # Phase 2 : préfixes cumulatifs → positions de sortie — O(k)
    for i in range(1, k):
        count[i] += count[i - 1]           # ✅ count[i] = nb d'éléments ≤ i

    # Phase 3 : placer chaque élément à sa position finale — O(n)
    output = [None] * len(arr)
    for x in reversed(arr):               # ✅ reversed pour stabilité
        idx = key(x) - min_val
        count[idx] -= 1
        output[count[idx]] = x

    return output
```

## Version simple (non stable, entiers seulement)

```python
def counting_sort_simple(arr):
    if not arr:
        return arr
    offset = min(arr)
    count  = [0] * (max(arr) - offset + 1)

    for x in arr:
        count[x - offset] += 1            # ✅ compter

    return [x + offset
            for x, c in enumerate(count)
            for _ in range(c)]            # ✅ reconstruire
```

## Illustration — étape par étape

```
Tableau : [4, 2, 2, 8, 3, 3, 1]
min=1, max=8, k=8

Phase 1 — count :
index : 0  1  2  3  4  5  6  7   (valeur = index+1)
count : 1  2  2  1  0  0  0  1

Phase 2 — cumulatif :
count : 1  3  5  6  6  6  6  7

Phase 3 — placement (reversed) :
  x=1 → count[0]=1→0, output[0]=1
  x=3 → count[2]=5→4, output[4]=3
  x=3 → count[2]=4→3, output[3]=3
  x=8 → count[7]=7→6, output[6]=8
  x=2 → count[1]=3→2, output[2]=2
  x=2 → count[1]=2→1, output[1]=2
  x=4 → count[3]=6→5, output[5]=4

output : [1, 2, 2, 3, 3, 4, 8] ✅
```

## Complexité

| Cas | Temps | Espace |
|-----|-------|--------|
| Best | O(n + k) | O(n + k) |
| Average | O(n + k) | O(n + k) |
| Worst | O(n + k) | O(n + k) |

## Quand counting sort est avantageux

```python
# ✅ Avantageux : n grand, k petit
notes = [14, 17, 12, 18, 14, 15, 17]    # k=20 (0..20), n=7
ages  = [25, 31, 19, 45, 25, 31]        # k=120, n raisonnable

# ❌ Inefficace : k >> n
ids   = [1000042, 9, 5000321, 42]        # k=5M pour 4 éléments → gaspillage
```

> [!tip] Counting sort comme sous-routine
> Counting sort est la brique de base du **tri radix** ([[Tri 06 — Tri par base (Radix Sort)]]). Sa stabilité est indispensable pour que radix sort fonctionne correctement.

> [!warning] Condition d'utilisation
> Réservé aux valeurs **entières et bornées**. Pour les flottants ou chaînes, utiliser bucket sort ou radix sort adapté. Si k > 10·n environ, le coût de l'initialisation du tableau de comptage dépasse le gain.
