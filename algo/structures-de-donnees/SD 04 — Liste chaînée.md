#algorithmique #structures-de-données #liste-chaînée #linked-list

## Principe
```
Chaînée simple :
┌───────┐   ┌───────┐   ┌───────┐
│val│next├──►│val│next├──►│val│null│
└───────┘   └───────┘   └───────┘
  head                    tail

Doublement chaînée :
null◄─┤prev│val│next├──►┤prev│val│next├──►null
         head                 tail
```

Chaque nœud contient une **valeur** et un **pointeur vers le suivant** (et le précédent pour la liste doublement chaînée).

## Interface abstraite
```
prepend(x)            → void         O(1)
append(x)             → void         O(n) simple / O(1) avec ptr tail
insert_after(node, x) → void         O(1) sur le pointeur
remove(x)             → bool         O(n)
contains(x)           → bool         O(n)
head()                → T | null
size()                → int
```

## Implémentation dans les langages courants

### Python
```python
from __future__ import annotations

class Node:
    def __init__(self, val: int) -> None:
        self.val  = val
        self.next: Node | None = None

class LinkedList:
    def __init__(self) -> None:
        self.head: Node | None = None

    def prepend(self, val: int) -> None:   # O(1)
        node      = Node(val)
        node.next = self.head
        self.head = node

    def to_list(self) -> list[int]:        # O(n)
        result, cur = [], self.head
        while cur:
            result.append(cur.val)
            cur = cur.next
        return result

# En pratique : collections.deque = liste doublement chaînée optimisée
from collections import deque
dll = deque([1, 2, 3])
dll.appendleft(0)   # O(1) tête
dll.append(4)       # O(1) queue
```

### Java
```java
// LinkedList<T> = liste doublement chaînée + Deque
LinkedList<Integer> ll = new LinkedList<>();
ll.addFirst(1);    // O(1) tête
ll.addLast(2);     // O(1) queue
ll.removeFirst();  // O(1)
ll.get(2);         // O(n) accès par index
```

### C++
```cpp
#include <list>
std::list<int> ll;
ll.push_front(1);     // O(1)
ll.push_back(2);      // O(1)
ll.insert(it, val);   // O(1) sur l'itérateur
ll.erase(it);         // O(1) sur l'itérateur
```

### JavaScript
```javascript
class ListNode {
  constructor(val, next = null) {
    this.val = val;
    this.next = next;
  }
}
```

## Complexités
| Opération | Liste chaînée | Tableau (array) |
|---|---|---|
| Accès par index | O(n) ❌ | O(1) ✅ |
| Insertion tête | O(1) ✅ | O(n) |
| Insertion queue | O(1) avec ptr tail | O(1) amorti |
| Insertion milieu (sur ptr) | O(1) ✅ | O(n) |
| Suppression tête | O(1) ✅ | O(n) |
| Recherche | O(n) | O(n) |
| Espace | O(n) + overhead ptr | O(n) |

## Algorithmes classiques sur listes chaînées

**Inverser une liste — O(n)**
```
prev = null, cur = head
tant que cur != null :
  next     = cur.next
  cur.next = prev
  prev     = cur
  cur      = next
head = prev
```

**Détecter un cycle — algorithme de Floyd**
```
slow = head, fast = head
tant que fast != null et fast.next != null :
  slow = slow.next
  fast = fast.next.next
  si slow is fast → cycle détecté   ← comparer les références, pas les valeurs
```

**Trouver le milieu — deux pointeurs**
```
slow = head, fast = head
tant que fast != null et fast.next != null :
  slow = slow.next
  fast = fast.next.next
→ slow est au milieu
```

**Fusionner deux listes triées — O(n+m)**
```
résultat = nœud sentinelle
cur = résultat
tant que l1 != null et l2 != null :
  si l1.val <= l2.val → cur.next = l1 ; l1 = l1.next
  sinon               → cur.next = l2 ; l2 = l2.next
  cur = cur.next
cur.next = l1 ou l2 (celle non épuisée)
```

> [!tip] LRU Cache = liste doublement chaînée + hash map
> Accès O(1) via le dict (clé → nœud), déplacement O(1) via les pointeurs.
> C'est exactement ce que `functools.lru_cache` (Python) et `LinkedHashMap` (Java) implémentent.

> [!warning] En pratique
> Les listes chaînées sont rarement utilisées directement — les tableaux dynamiques (`list`, `ArrayList`, `vector`) ont de meilleures performances cache CPU.
> Elles restent indispensables dans les entretiens techniques et pour comprendre `deque`, `LinkedHashMap`, et les caches LRU.
