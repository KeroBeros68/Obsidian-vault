#algorithmique #structures-de-données #tas #heap #min-heap #max-heap

## Principe
```
Arbre binaire presque complet où chaque parent ≤ ses enfants (min-heap)
ou chaque parent ≥ ses enfants (max-heap).

Min-heap :        Max-heap :
      1                 9
     / \               / \
    3   5             7   8
   / \ / \           / \ /
  4  8 6  9         3  5 6
```

**Propriété** : la racine est toujours le minimum (min-heap) ou le maximum (max-heap).

## Représentation en tableau — sans pointeurs
```
Tableau :  [1, 3, 5, 4, 8, 6, 9]
Indices :   0  1  2  3  4  5  6

Parent de i        = (i - 1) / 2
Enfant gauche de i = 2*i + 1
Enfant droit  de i = 2*i + 2

Racine = index 0  →  toujours le minimum (min-heap)
```

## Opérations fondamentales

**heapify-up (sift-up) — après insertion**
```
Insérer à la fin du tableau
Tant que nœud < parent → échanger avec le parent (remonter)
```

**heapify-down (sift-down) — après extraction**
```
Remplacer la racine par le dernier élément, supprimer le dernier
Tant que nœud > un enfant → échanger avec le plus petit enfant (descendre)
```

**Construire un tas depuis un tableau — O(n)**
```
Partir du dernier nœud non-feuille : i = n/2 - 1
Pour i de n/2-1 à 0 : sift-down(i)
(Plus rapide que n insertions qui donneraient O(n log n))
```

## Implémentation dans les langages courants

### Python
```python
import heapq   # min-heap uniquement

data = [5, 3, 8, 1, 9]
heapq.heapify(data)           # O(n) — en place

heapq.heappush(data, 4)       # O(log n)
minimum = heapq.heappop(data) # O(log n)
minimum = data[0]             # peek O(1)

# Max-heap : stocker les négatifs
max_heap = [-x for x in data]
heapq.heapify(max_heap)
maximum = -heapq.heappop(max_heap)

# N plus petits / grands
heapq.nsmallest(3, data)      # O(n log k)
heapq.nlargest(3, data)
```

### Java
```java
// Min-heap
PriorityQueue<Integer> minH = new PriorityQueue<>();
minH.offer(5); minH.offer(1); minH.offer(3);
minH.poll();   // 1
minH.peek();   // prochain min

// Max-heap
PriorityQueue<Integer> maxH =
    new PriorityQueue<>(Collections.reverseOrder());
```

### C++
```cpp
#include <queue>
#include <vector>
#include <algorithm>

// Max-heap (défaut STL)
std::priority_queue<int> maxH;
maxH.push(5); maxH.top(); maxH.pop();

// Min-heap
std::priority_queue<int,std::vector<int>,std::greater<int>> minH;

// Construire depuis un tableau
std::vector<int> v = {5,3,8,1};
std::make_heap(v.begin(), v.end());   // max-heap O(n)
```

### JavaScript / TypeScript
```typescript
class MinHeap {
  private data: number[] = [];

  push(val: number): void {
    this.data.push(val);
    this._siftUp(this.data.length - 1);
  }
  pop(): number | undefined {
    if (!this.data.length) return undefined;
    const min = this.data[0];
    const last = this.data.pop()!;
    if (this.data.length) { this.data[0] = last; this._siftDown(0); }
    return min;
  }
  peek(): number | undefined { return this.data[0]; }
  get size(): number         { return this.data.length; }

  private _siftUp(i: number): void {
    while (i > 0) {
      const parent = (i - 1) >> 1;
      if (this.data[parent] <= this.data[i]) break;
      [this.data[parent], this.data[i]] = [this.data[i], this.data[parent]];
      i = parent;
    }
  }
  private _siftDown(i: number): void {
    const n = this.data.length;
    while (true) {
      let min = i;
      const l = 2*i+1, r = 2*i+2;
      if (l < n && this.data[l] < this.data[min]) min = l;
      if (r < n && this.data[r] < this.data[min]) min = r;
      if (min === i) break;
      [this.data[min], this.data[i]] = [this.data[i], this.data[min]];
      i = min;
    }
  }
}
```

## Complexités
| Opération | Complexité |
|---|---|
| peek (min/max) | O(1) |
| push | O(log n) |
| pop | O(log n) |
| heapify (n éléments) | O(n) |
| nsmallest(k, n) | O(n log k) |
| Espace | O(n) |

## Tri par tas — Heap Sort
```
1. Construire un max-heap : O(n)
2. Pour i de n-1 à 1 :
     échanger racine (max) avec l'élément en position i
     sift-down sur les i premiers éléments
   → O(n log n)

Complexité totale : O(n log n) — en place, pas stable
```

> [!tip] Tas vs tableau trié
> `sorted()` pour un accès ordonné statique.
> `heapq` / PriorityQueue pour extraire dynamiquement le min/max après des insertions.
> Si k << n : `nsmallest(k)` avec un tas est O(n log k) — mieux que O(n log n) pour trier tout.
