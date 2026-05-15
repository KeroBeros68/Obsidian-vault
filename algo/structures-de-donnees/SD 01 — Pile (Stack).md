#algorithmique #structures-de-données #pile #stack #lifo

## Principe — LIFO
```
Last In, First Out — Dernier entré, premier sorti

   push →   ┌───┐
	        │ C │  ← sommet (top)
            │ B │
            │ A │  ← base
            └───┘
              ↑ pop
```

Les deux opérations fondamentales s'effectuent toutes les deux **au sommet** :
- **push(x)** — empiler un élément
- **pop()** — dépiler et retourner l'élément du sommet
- **peek()** — consulter le sommet sans le retirer

## Interface abstraite
```
push(x)       → void
pop()         → T          (erreur si vide)
peek()        → T          (erreur si vide)
is_empty()    → bool
size()        → int
```

## Implémentation dans les langages courants

### Python
```python
# list native — O(1) amorti push/pop
stack = []
stack.append(3)    # push
stack.pop()        # pop
stack[-1]          # peek

# Ou collections.deque (thread-safe)
from collections import deque
stack = deque()
stack.append(3)
stack.pop()
```

### JavaScript / TypeScript
```typescript
const stack: number[] = [];
stack.push(3);           // push
stack.pop();             // pop
stack[stack.length - 1]; // peek
```

### Java
```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(3);           // push
stack.pop();             // pop
stack.peek();            // peek
// Stack<T> existe mais est deprecated — préférer ArrayDeque
```

### C++
```cpp
#include <stack>
std::stack<int> st;
st.push(3);    // push
st.pop();      // pop (void — ne retourne pas)
st.top();      // peek
```

### Go
```go
// Pas de Stack natif — utiliser une slice
stack := []int{}
stack = append(stack, 3)          // push
top := stack[len(stack)-1]        // peek
stack = stack[:len(stack)-1]      // pop
```

## Complexités
| Opération | Complexité |
|---|---|
| push | O(1) amorti |
| pop | O(1) |
| peek | O(1) |
| is_empty | O(1) |
| Espace | O(n) |

## Cas d'usage classiques

**1. Vérification de parenthèses équilibrées**
```
"(a[b]c)"  → ✅
"(a[b)c]"  → ❌

Algorithme :
  pour chaque caractère :
    si ouvrant → push
    si fermant → pop et vérifier la correspondance
  à la fin → pile vide = équilibré
```

**2. DFS itératif**
```
Initialiser : push(nœud_départ)
Boucle :
  node = pop()
  si non visité → marquer + traiter
  pour chaque voisin non visité → push(voisin)
```

**3. Évaluation d'expressions (notation polonaise inverse)**
```
"3 4 + 2 *"  →  (3 + 4) * 2 = 14

Algorithme :
  pour chaque token :
    si nombre → push
    si opérateur → pop deux valeurs, calculer, push résultat
```

**4. Historique / undo-redo**
```
Undo stack : push à chaque action, pop pour annuler
Redo stack : push le pop d'undo, pop pour refaire
```

> [!tip] Pile = récursion itérative
> Toute fonction récursive peut être réécrite avec une pile explicite.
> Utile pour éviter les stack overflow sur de grandes profondeurs.
