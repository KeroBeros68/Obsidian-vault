#algorithmique #structures-de-données #abr #bst #arbre #binaire

## Propriété fondamentale
```
Pour tout nœud N :
  sous-arbre GAUCHE  → toutes valeurs < N
  sous-arbre DROIT   → toutes valeurs > N

        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4   7 13

Parcours infixe (in-order) → valeurs triées : 1 3 4 6 7 8 10 13 14
```

## Interface abstraite
```
insert(x)       → void
contains(x)     → bool
remove(x)       → bool
min()           → T
max()           → T
inorder()       → List<T>   (valeurs triées)
height()        → int
```

## Algorithmes fondamentaux

**Insertion**
```
si nœud == null → créer feuille
si x < nœud.val → insérer dans sous-arbre gauche
si x > nœud.val → insérer dans sous-arbre droit
si x == nœud.val → doublon, ignorer (ou compter)
```

**Recherche**
```
si nœud == null     → non trouvé
si x == nœud.val    → trouvé
si x < nœud.val     → chercher à gauche
si x > nœud.val     → chercher à droite
```

**Suppression — 3 cas**
```
Cas 1 — feuille         : supprimer directement
Cas 2 — un enfant       : remplacer par l'enfant
Cas 3 — deux enfants    : remplacer par le successeur infixe
                          (min du sous-arbre droit)
                          puis supprimer ce successeur
```

## Les trois parcours
```
Infixe  (in-order)  : gauche → racine → droite  → trié
Préfixe (pre-order) : racine → gauche → droite  → sérialisation
Suffixe (post-order): gauche → droite → racine  → suppression sûre

Largeur (BFS)       : niveau par niveau            → file FIFO
```

## Implémentation dans les langages courants

### Python
```python
from __future__ import annotations

class BSTNode:
    def __init__(self, val: int) -> None:
        self.val   = val
        self.left  = self.right = None

def insert(root: BSTNode | None, val: int) -> BSTNode:
    if root is None:        return BSTNode(val)
    if val < root.val:      root.left  = insert(root.left,  val)
    elif val > root.val:    root.right = insert(root.right, val)
    return root

def inorder(root: BSTNode | None) -> list[int]:
    if root is None: return []
    return inorder(root.left) + [root.val] + inorder(root.right)

# ABR équilibré — SortedList (O(log n) partout)
from sortedcontainers import SortedList
sl = SortedList([5, 3, 8, 1])
sl.add(4)          # O(log n)
sl.remove(3)       # O(log n)
sl[0]              # minimum O(log n)
3 in sl            # O(log n)
```

### Java
```java
// TreeMap = ABR rouge-noir équilibré
TreeMap<Integer, String> map = new TreeMap<>();
map.put(5, "cinq");
map.firstKey();    // min O(log n)
map.lastKey();     // max O(log n)
map.floorKey(4);   // plus grand ≤ 4
map.ceilingKey(4); // plus petit ≥ 4

// TreeSet = ABR sans valeurs
TreeSet<Integer> set = new TreeSet<>();
set.add(3); set.add(1); set.add(5);
set.first();     // 1
set.last();      // 5
```

### C++
```cpp
#include <set>
#include <map>
// std::set / std::map = ABR rouge-noir (balanced)
std::set<int> s = {5, 3, 8, 1};
s.insert(4);
auto it = s.begin();   // min
*s.rbegin();           // max
s.lower_bound(4);      // premier ≥ 4
```

### JavaScript / TypeScript
```typescript
// Pas d'ABR natif — implémenter ou utiliser une lib
// (sorted-btree, functional-red-black-tree)
```

## Complexités
| Opération | ABR moyen (équilibré) | ABR pire cas (dégénéré) |
|---|---|---|
| insert | O(log n) | O(n) |
| contains | O(log n) | O(n) |
| remove | O(log n) | O(n) |
| min / max | O(log n) | O(n) |
| inorder | O(n) | O(n) |
| Hauteur | O(log n) | O(n) |

## Arbres auto-équilibrés — O(log n) garanti
| Arbre | Langages standards |
|---|---|
| AVL | implémentations custom |
| Rouge-Noir | Java `TreeMap/TreeSet`, C++ `std::map/set`, Rust `BTreeMap` |
| B-Tree | bases de données (PostgreSQL, SQLite) |
| Skip List | Redis, Java `ConcurrentSkipListMap` |

> [!warning] ABR dégénéré
> Insertions en ordre croissant 1, 2, 3, 4... → arbre = liste chaînée → O(n) partout.
> En production : toujours utiliser un arbre auto-équilibré (`TreeMap`, `std::map`, `SortedList`).
