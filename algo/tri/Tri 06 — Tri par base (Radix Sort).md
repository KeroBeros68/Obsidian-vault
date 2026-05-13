#algo #tri #radix #linéaire

## Tri radix — Radix Sort

Trie les entiers **chiffre par chiffre**, du chiffre le moins significatif au plus significatif (LSD), en utilisant un tri stable (counting sort) à chaque passe. O(d·n) sans comparaisons.

## Propriétés

- **O(d · (n + b))** où d = nombre de chiffres, b = base (souvent 10 ou 256).
- **Stable** : préserve l'ordre relatif à chaque passe (condition critique pour la correction).
- Pas de comparaisons entre éléments — brise la borne Ω(n log n).
- Efficace quand d est petit (entiers de taille fixe : int32, int64).

## Deux variantes

| Variante | Ordre de traitement | Usage |
|----------|--------------------|----|
| **LSD** (Least Significant Digit) | chiffre des unités → dizaines → ... | Entiers de longueur fixe ✅ |
| **MSD** (Most Significant Digit) | chiffre de tête → ... | Chaînes, longueurs variables |

## Implémentation LSD

```python
def radix_sort(arr, base=10):
    if not arr:
        return arr

    max_val = max(arr)
    exp     = 1                             # ✅ commencer par les unités

    while max_val // exp > 0:
        arr = counting_sort_by_digit(arr, exp, base)
        exp *= base

    return arr

def counting_sort_by_digit(arr, exp, base):
    n      = len(arr)
    output = [0] * n
    count  = [0] * base

    # Compter les occurrences du chiffre courant
    for x in arr:
        digit = (x // exp) % base
        count[digit] += 1

    # Préfixes cumulatifs
    for i in range(1, base):
        count[i] += count[i - 1]

    # Placement en ordre inverse pour stabilité ✅
    for x in reversed(arr):
        digit         = (x // exp) % base
        count[digit] -= 1
        output[count[digit]] = x

    return output
```

## Illustration — 3 passes sur des entiers 3 chiffres

```
Initial  : [170, 045, 075, 090, 002, 024, 802, 066]

Passe 1 (unités) :
  170→0  090→0  002→2  802→2  024→4  045→5  075→5  066→6
  → [170, 090, 002, 802, 024, 045, 075, 066]

Passe 2 (dizaines) :
  002→0  802→0  024→2  045→4  066→6  170→7  075→7  090→9
  → [002, 802, 024, 045, 066, 170, 075, 090]

Passe 3 (centaines) :
  002→0  024→0  045→0  066→0  075→0  090→0  170→1  802→8
  → [002, 024, 045, 066, 075, 090, 170, 802] ✅

La stabilité à chaque passe garantit le bon résultat final.
```

## Pourquoi la stabilité est indispensable

```python
# Après passe 1 (tri par unités) :
# [170, 090, 002, 802, ...]
#  ↑                ↑
#  Ces deux sont déjà dans le bon ordre relatif (0...0, puis 2...2)
# Si la passe 2 n'est PAS stable, elle peut réordonner 002 et 802
# → la passe 3 trierait mal les centaines. ❌

# Avec counting sort stable : l'ordre établi aux passes précédentes
# est préservé → résultat final correct. ✅
```

## Radix sort sur chaînes (MSD)

```python
def radix_sort_strings(arr, pos=0):
    if len(arr) <= 1:
        return arr

    buckets = {}
    for s in arr:
        char = s[pos] if pos < len(s) else ''
        buckets.setdefault(char, []).append(s)

    result = []
    for key in sorted(buckets.keys()):          # ✅ ordre lexicographique
        group = buckets[key]
        if key == '':
            result.extend(group)                # ✅ chaînes courtes en premier
        else:
            result.extend(radix_sort_strings(group, pos + 1))

    return result
```

## Complexité

| Cas | Temps | Espace |
|-----|-------|--------|
| Best | O(d · (n + b)) | O(n + b) |
| Average | O(d · (n + b)) | O(n + b) |
| Worst | O(d · (n + b)) | O(n + b) |

```
Pour des int32 en base 256 : d = 4 passes, b = 256
→ 4 · (n + 256) ≈ 4n pour n grand
→ Très compétitif face à O(n log n) pour n > ~100 000
```

## Comparaison counting vs radix

| Critère | Counting Sort | Radix Sort |
|---------|--------------|-----------|
| Valeurs bornées petites (k ≤ 1000) | ✅ direct | ⚠️ overhead de d passes |
| Grands entiers (int32, int64) | ❌ k trop grand | ✅ d fixe (4 ou 8 passes) |
| Chaînes | ❌ | ✅ MSD |
| Stabilité | ✅ | ✅ (si counting stable) |

> [!tip] Radix sort en base 256 pour les entiers machine
> Pour des int32 : 4 passes en base 256 (octets). Pour des int64 : 8 passes. C'est la version la plus rapide en pratique — comparable à std::sort sur de grands tableaux.

> [!warning] Entiers négatifs
> L'implémentation naïve ne gère pas les négatifs. Solution : décaler tous les entiers de `|min|` avant le tri, puis redécaler après. Ou traiter le dernier octet (signe) séparément.

> [!info] Lien avec counting sort
> Radix sort utilise counting sort comme sous-routine à chaque passe. La stabilité de counting sort est **la condition sine qua non** du bon fonctionnement de radix sort.
