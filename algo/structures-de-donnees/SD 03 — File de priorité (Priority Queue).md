#algorithmique #structures-de-données #priority-queue #heap #dijkstra

## Principe
```
Chaque élément a une priorité.
dequeue() retourne toujours l'élément de priorité minimale (ou maximale).

  enqueue(tâche, priorité)
  dequeue() → tâche de plus haute priorité

  ┌────────────────┐
  │ (1) Urgence    │  ← toujours sorti en premier
  │ (3) Normal     │
  │ (5) Basse prio │
  └────────────────┘
```

Implémentée en interne par un **tas binaire** — voir [[SD 05 — Tas binaire (Heap)]].

## Interface abstraite
```
push(x, priority)   → void
pop()               → T      (retire et retourne le min/max)
peek()              → T      (consulte sans retirer)
is_empty()          → bool
size()              → int
```

## Implémentation dans les langages courants

### Python — heapq (min-heap)
```python
import heapq

pq = []
heapq.heappush(pq, (1, "urgent"))
heapq.heappush(pq, (3, "normal"))
heapq.heappush(pq, (5, "basse prio"))

priority, task = heapq.heappop(pq)   # (1, "urgent")
peek = pq[0]                          # sans retirer

# Tie-breaker pour éléments non comparables
from itertools import count
ctr = count()
heapq.heappush(pq, (1, next(ctr), obj_a))
heapq.heappush(pq, (1, next(ctr), obj_b))
```

### Java
```java
// Min-heap par défaut
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(3);
pq.offer(1);
pq.poll();    // 1 (min)
pq.peek();    // prochain min sans retirer

// Max-heap
PriorityQueue<Integer> maxPQ =
    new PriorityQueue<>(Collections.reverseOrder());

// Avec comparateur custom
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
```

### C++
```cpp
#include <queue>
// Max-heap par défaut
std::priority_queue<int> pq;
pq.push(3);
pq.top();     // max
pq.pop();

// Min-heap
std::priority_queue<int, std::vector<int>, std::greater<int>> minPQ;
```

### JavaScript / TypeScript
```typescript
// Pas de PQ native — implémenter un min-heap
class MinHeap<T> {
  private data: [number, T][] = [];
  push(priority: number, item: T): void { /* heapify up */ }
  pop(): T | undefined { /* heapify down */ }
  peek(): T | undefined { return this.data[0]?.[1]; }
}
```

### Go
```go
// Implémenter heap.Interface
import "container/heap"

type MinHeap []int
func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MinHeap) Push(x any)        { *h = append(*h, x.(int)) }
func (h *MinHeap) Pop() any          { /* retirer le dernier */ }
```

## Complexités
| Opération | Complexité |
|---|---|
| push | O(log n) |
| pop (min/max) | O(log n) |
| peek | O(1) |
| construction depuis n éléments | O(n) |
| Espace | O(n) |

## Cas d'usage classiques

**Dijkstra — plus court chemin**
```
pq = [(0, source)]
tant que pq non vide :
  (dist, u) = pq.pop_min()
  pour chaque (v, poids) voisin de u :
    si dist + poids < dist[v] :
      dist[v] = dist + poids
      pq.push((dist[v], v))
```
→ [[Graphes 05 — Dijkstra — Plus court chemin]]

**A* — recherche heuristique**
```
pq = [(h(start), start)]  où h = heuristique
tant que pq non vide :
  (f, u) = pq.pop_min()   f = g + h
  ...
```
→ [[Graphes 06 — A-star — Recherche heuristique]]

**Prim — arbre couvrant minimal**
```
pq = [(0, source, -1)]
tant que pq non vide :
  (poids, u, parent) = pq.pop_min()
  si u non visité → ajouter arête (parent, u) au MST
  pour chaque (v, w) voisin de u :
    pq.push((w, v, u))
```
→ [[Graphes 08 — Prim — Arbre couvrant minimal]]

**Ordonnanceur de tâches, top-k éléments, flux de médiane**

> [!warning] Pas de decrease-key dans la plupart des implémentations
> `heapq` (Python), `PriorityQueue` (Java), `priority_queue` (C++) ne supportent pas la modification de priorité.
> Solution standard : **lazy deletion** — pousser une nouvelle entrée avec la nouvelle priorité, ignorer les obsolètes via un set `visited` à l'extraction.

> [!important] Connexion graphes
> La file de priorité est la structure centrale de **Dijkstra**, **A\*** et **Prim**.
> → [[Graphes 05 — Dijkstra — Plus court chemin]], [[Graphes 06 — A-star — Recherche heuristique]], [[Graphes 08 — Prim — Arbre couvrant minimal]]
