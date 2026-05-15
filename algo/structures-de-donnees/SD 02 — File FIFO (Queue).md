#algorithmique #structures-de-données #file #queue #fifo

## Principe — FIFO
```
First In, First Out — Premier entré, premier sorti

enqueue → ┌───┬───┬───┐ → dequeue
          │ C │ B │ A │
          └───┴───┴───┘
           queue         tête
```

- **enqueue(x)** — ajouter en queue
- **dequeue()** — retirer et retourner depuis la tête
- **peek()** — consulter la tête sans retirer

## Interface abstraite
```
enqueue(x)    → void
dequeue()     → T     (erreur si vide)
peek()        → T     (erreur si vide)
is_empty()    → bool
size()        → int
```

## Implémentation dans les langages courants

### Python
```python
from collections import deque          # O(1) des deux côtés
q = deque()
q.append("A")       # enqueue
q.popleft()         # dequeue — O(1) ✅
q[0]                # peek

# ❌ Ne pas utiliser list.pop(0) → O(n)
```

### JavaScript / TypeScript
```typescript
// Pas de Queue native O(1) — simuler avec un tableau + index de tête
class Queue<T> {
  private data: T[] = [];
  private head = 0;
  enqueue(x: T): void { this.data.push(x); }
  dequeue(): T | undefined {
    if (this.head >= this.data.length) return undefined;
    return this.data[this.head++];
  }
  peek(): T | undefined { return this.data[this.head]; }
  get size(): number { return this.data.length - this.head; }
}
```

### Java
```java
Queue<Integer> q = new ArrayDeque<>();   // ArrayDeque plus performant que LinkedList
q.offer(3);      // enqueue
q.poll();        // dequeue (null si vide)
q.peek();        // peek    (null si vide)
```

### C++
```cpp
#include <queue>
std::queue<int> q;
q.push(3);     // enqueue
q.pop();       // dequeue (void)
q.front();     // peek
```

### Go
```go
// Pas de Queue native — utiliser une slice ou un channel
queue := []int{}
queue = append(queue, 3)          // enqueue
front := queue[0]                 // peek
queue = queue[1:]                 // dequeue — O(n) ! Préférer list.List
```

## File double — Deque
```
Insertions et suppressions O(1) des deux côtés.

appendleft  ←  ┌───┬───┬───┐  ← append
               │ A │ B │ C │
               └───┴───┴───┘
popleft    →                  → pop
```

| Langage | Type |
|---|---|
| Python | `collections.deque` |
| Java | `ArrayDeque<T>` |
| C++ | `std::deque<T>` |
| C# | `LinkedList<T>` |
| JS | Pas de natif — implémenter |

## Complexités
| Opération | list / array | deque / linked |
|---|---|---|
| enqueue (queue) | O(1) amorti | O(1) |
| dequeue (tête) | O(n) ❌ | O(1) ✅ |
| peek | O(1) | O(1) |
| Accès par index | O(1) | O(n) |

## Cas d'usage classiques

**BFS — parcours en largeur**
```
Initialiser : enqueue(nœud_départ), marquer visité
Boucle :
  node = dequeue()
  traiter node
  pour chaque voisin non visité :
    marquer visité
    enqueue(voisin)
```

**Fenêtre glissante / buffer circulaire**
```
Garder les N derniers événements :
→ deque(maxlen=N) en Python
→ buffer circulaire de taille N sinon
```

> [!important] Connexion graphes
> La file FIFO est la structure centrale de **BFS** (parcours en largeur).
> → [[Graphes 02 — BFS — Parcours en largeur]]
