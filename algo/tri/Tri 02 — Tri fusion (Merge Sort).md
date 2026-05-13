#algo #tri #fusion #diviser-pour-régner

## Tri fusion — Merge Sort

Divise le tableau en deux moitiés, trie chacune récursivement, puis **fusionne** les deux moitiés triées. Paradigme **diviser pour régner**.

## Propriétés

- Complexité **garantie** O(n log n) dans tous les cas — pas de worst case dégradé.
- **Stable** : les éléments égaux conservent leur ordre relatif.
- Nécessite O(n) espace auxiliaire — seul vrai inconvénient.
- Naturellement adapté aux **listes chaînées** (pas de random access requis).

## Implémentation

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr                          # ✅ cas de base

    mid   = len(arr) // 2
    left  = merge_sort(arr[:mid])           # ✅ diviser
    right = merge_sort(arr[mid:])

    return merge(left, right)               # ✅ fusionner

def merge(left, right):
    result = []
    i = j  = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:             # ✅ <= pour garantir la stabilité
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])                 # ✅ éléments restants
    result.extend(right[j:])
    return result
```

## Version en place (optimisée mémoire)

```python
def merge_sort_inplace(arr, left=0, right=None):
    if right is None:
        right = len(arr) - 1
    if left >= right:
        return

    mid = (left + right) // 2
    merge_sort_inplace(arr, left, mid)
    merge_sort_inplace(arr, mid + 1, right)
    merge_inplace(arr, left, mid, right)

def merge_inplace(arr, left, mid, right):
    tmp = arr[left:right+1]                 # ⚠️ copie locale O(n) — inévitable
    i, j, k = 0, mid - left + 1, left
    while i <= mid - left and j <= right - left:
        if tmp[i] <= tmp[j]:
            arr[k] = tmp[i]; i += 1
        else:
            arr[k] = tmp[j]; j += 1
        k += 1
    while i <= mid - left:
        arr[k] = tmp[i]; i += 1; k += 1
    while j <= right - left:
        arr[k] = tmp[j]; j += 1; k += 1
```

## Illustration — arbre de récursion

```
[5, 2, 8, 1, 9, 3]
        ↙           ↘
  [5, 2, 8]       [1, 9, 3]
   ↙      ↘        ↙      ↘
 [5]   [2, 8]   [1]    [9, 3]
        ↙  ↘            ↙  ↘
       [2] [8]          [9] [3]

Fusion remontante :
[2,8] → [2,5,8] → ...
[3,9] → [1,3,9] → ...
                  → [1,2,3,5,8,9] ✅

Profondeur : log₂(6) ≈ 3 niveaux
Travail par niveau : O(n)
Total : O(n log n)
```

## Complexité détaillée

| Cas | Comparaisons | Espace |
|-----|-------------|--------|
| Best | O(n log n) | O(n) |
| Average | O(n log n) | O(n) |
| Worst | O(n log n) | O(n) |

```
Récurrence : T(n) = 2·T(n/2) + O(n)
Théorème maître (cas 2) : a=2, b=2, f(n)=n → T(n) = O(n log n) ✅
```

> [!tip] Fusion de listes chaînées
> Le tri fusion est optimal sur les listes chaînées : pas besoin d'accès aléatoire, et la fusion se fait en O(n) avec O(1) espace supplémentaire (relier les nœuds, pas de copie).

> [!warning] Espace O(n)
> Le tri fusion ne peut pas être fait en vrai O(1) espace tout en restant O(n log n). Les implémentations "en place" copient quand même un sous-tableau localement.

> [!info] Base du Timsort
> Timsort (Python, Java) est une optimisation du tri fusion qui détecte les **runs** (sous-séquences déjà triées) et fusionne intelligemment. O(n) sur données quasi-triées.
