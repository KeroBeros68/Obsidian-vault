#algo #arbres #heap #heapq #python #avancé

## Rappel : heapq = min-heap uniquement

```python
import heapq
data = [5, 3, 8, 1]
heapq.heapify(data)          # O(n) — transforme la liste en min-heap en place
heapq.heappush(data, 4)      # O(log n)
heapq.heappop(data)          # O(log n) → retire et retourne le minimum
data[0]                      # O(1) — peek sans retirer
```

Pour les bases (structure, heapify-up/down, implémentation multi-langages) : voir [[SD 05 — Tas binaire (Heap)]].

## Max-heap avec heapq

```python
# Inverser le signe — seule solution native Python
max_heap = [-x for x in [5, 3, 8, 1]]
heapq.heapify(max_heap)
maximum = -heapq.heappop(max_heap)   # ✅

# Stocker (−priorité, objet) pour des tuples
tasks = [(-10, "haute"), (-3, "basse")]
heapq.heapify(tasks)
_, task = heapq.heappop(tasks)       # "haute" ✅
```

## Objets personnalisés dans le heap

```python
from dataclasses import dataclass, field
import itertools

# Option 1 — tuple (priorité, objet) : simple mais crashe si priorités égales
#            et que les objets ne sont pas comparables

# Option 2 — tie-breaker avec compteur (recommandé)
counter = itertools.count()

def heap_push(heap, priority, obj):
    heapq.heappush(heap, (priority, next(counter), obj))  # ✅ jamais de comparaison sur obj

# Option 3 — dataclass avec order=True
@dataclass(order=True)
class Task:
    priority: int
    name: str = field(compare=False)   # exclure name du tri
```

## heappushpop et heapreplace

```python
# heappushpop — push + pop en une seule opération, plus rapide
result = heapq.heappushpop(heap, item)
# Garantit : result ≤ item si heap non vide
# Utile pour maintenir un heap de taille fixe

# heapreplace — retire la racine PUIS insère (contrairement à heappushpop)
result = heapq.heapreplace(heap, item)
# ⚠️ raise IndexError si heap vide (heappushpop ne raise pas)
# Légèrement plus rapide que heappushpop si item > racine actuelle
```

## nsmallest / nlargest — k éléments extrêmes

```python
heapq.nsmallest(3, data)   # O(n log k) — fonctionne sur tout itérable
heapq.nlargest(3, data)    # O(n log k)

# Règle de performance :
# k == 1    → min(data) / max(data)         O(n)       ← plus rapide
# k ≈ n     → sorted(data)[:k]             O(n log n) ← plus rapide
# 1 < k << n → heapq.nsmallest / nlargest  O(n log k) ✅
```

## merge — fusion de k listes triées

```python
lists = [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
merged = list(heapq.merge(*lists))   # [1, 2, 3, 4, 5, 6, 7, 8, 9]
# O(n log k) — n éléments au total, k listes
# ✅ Paresseux (générateur) — ne charge pas tout en mémoire
```

## Pattern — K-ième plus grand élément

```python
def kth_largest(nums: list[int], k: int) -> int:
    # Maintenir un min-heap de taille k
    # → la racine est le k-ième plus grand element
    heap = nums[:k]
    heapq.heapify(heap)                       # O(k)
    for n in nums[k:]:
        if n > heap[0]:
            heapq.heapreplace(heap, n)        # O(log k)
    return heap[0]
# Complexité : O(n log k) temps, O(k) espace
```

## Tableau de synthèse

| Opération | Complexité | Note |
|---|---|---|
| heapify(n) | O(n) | En place |
| heappush | O(log n) | |
| heappop | O(log n) | |
| heappushpop | O(log n) | Plus rapide que push + pop séparés |
| heapreplace | O(log n) | Raise IndexError si vide |
| nsmallest(k, n) | O(n log k) | Optimal si 1 < k << n |
| nlargest(k, n) | O(n log k) | Optimal si 1 < k << n |
| merge(k listes, n) | O(n log k) | Générateur paresseux |

> [!tip] heapq vs [[SD 03 — File de priorité (Priority Queue)|PriorityQueue]]
> `heapq` est non thread-safe et légèrement plus rapide (pas de verrou).
> `queue.PriorityQueue` est thread-safe (synchronisé) — contexte concurrent multi-thread.
> `sortedcontainers.SortedList` si tu as besoin d'accès par rang (index[i]) en O(log n).

> [!warning] Objets non comparables — crash silencieux
> `heapq.heappush(heap, (5, obj))` crashe avec `TypeError` si deux tuples ont la même priorité
> et que `obj` n'implémente pas `__lt__`.
> Toujours ajouter un tie-breaker `itertools.count()` entre priorité et objet.
