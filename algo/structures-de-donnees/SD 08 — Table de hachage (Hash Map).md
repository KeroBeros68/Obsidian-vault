#algorithmique #structures-de-données #hash #hashmap #hashset #hachage

## Principe
```
clé → fonction de hachage → index dans un tableau → valeur

"Alice" → hash("Alice") % capacité → tableau[42] → 30

Collision : deux clés différentes → même index
  → Chaînage       : liste à cet index
  → Adressage ouvert : chercher la prochaine case libre
```

## Interface abstraite
```
put(key, value)    → void
get(key)           → V | null
remove(key)        → bool
contains_key(key)  → bool
keys()             → Set<K>
size()             → int
```

## Implémentation dans les langages courants

### Python
```python
# dict — hash map native, O(1) moyen
d: dict[str, int] = {}
d["Alice"]  = 30           # put
d.get("Bob", 0)            # get avec défaut
del d["Alice"]             # remove
"Alice" in d               # contains_key — O(1)

# Fusion (Python 3.9+)
merged = d1 | d2

# defaultdict — valeur par défaut automatique
from collections import defaultdict
graph = defaultdict(list)
graph[1].append(2)         # pas de KeyError

# Counter — comptage
from collections import Counter
c = Counter("abracadabra")
c.most_common(3)           # [("a",5), ("b",2), ("r",2)]

# set — hash set
s: set[str] = {"Alice", "Bob"}
s.add("Charlie")           # O(1)
"Alice" in s               # O(1) — ×10-100 plus rapide que list
```

### Java
```java
Map<String,Integer> map = new HashMap<>();
map.put("Alice", 30);
map.get("Alice");              // 30
map.getOrDefault("Bob", 0);   // 0
map.containsKey("Alice");      // true
map.remove("Alice");

// LinkedHashMap — conserve l'ordre d'insertion
Map<String,Integer> lhm = new LinkedHashMap<>();

// TreeMap — trié par clé, O(log n)
Map<String,Integer> tm = new TreeMap<>();

// HashSet<T>
Set<String> set = new HashSet<>();
set.add("Alice");
set.contains("Alice");   // O(1)
```

### C++
```cpp
#include <unordered_map>
#include <unordered_set>

std::unordered_map<std::string,int> umap;
umap["Alice"] = 30;
umap.count("Alice");     // 0 ou 1
umap.erase("Alice");

std::unordered_set<std::string> uset;
uset.insert("Alice");
uset.count("Alice");     // O(1) moyen

// Trié : std::map / std::set (O(log n))
```

### JavaScript / TypeScript
```typescript
const map = new Map<string, number>();
map.set("Alice", 30);
map.get("Alice");          // 30
map.has("Alice");          // true
map.delete("Alice");

const set = new Set<string>();
set.add("Alice");
set.has("Alice");          // true
```

### Go
```go
m := map[string]int{}
m["Alice"] = 30
val, ok := m["Alice"]   // ok = false si absent
delete(m, "Alice")
```

## Complexités
| Opération | Moyen | Pire cas |
|---|---|---|
| put | O(1) | O(n) |
| get | O(1) | O(n) |
| remove | O(1) | O(n) |
| contains | O(1) | O(n) |
| Espace | O(n) | O(n) |

Le pire cas O(n) survient en cas de collisions massives (mauvaise fonction de hachage ou attaque DoS).

## Conditions de hashabilité
```
Un objet utilisable comme clé doit :
  1. Implémenter une fonction de hachage
  2. Implémenter l'égalité (==)
  3. Être cohérent : si a == b alors hash(a) == hash(b)
  4. Être stable : hash(a) ne change pas pendant la durée de vie de a

Python  : __hash__ + __eq__    — list/dict ne sont PAS hashables
Java    : hashCode() + equals()
C++     : std::hash<T> + operator==
```

## Gestion des collisions — deux stratégies
```
Chaînage (Chaining) :
  tableau[i] = liste de (clé, valeur)
  Avantage : simple, pas de limite de charge
  Java HashMap, Python dict (historiquement)

Adressage ouvert (Open Addressing) :
  En cas de collision → chercher la prochaine case libre
  Sondage linéaire : i+1, i+2, ...
  Sondage quadratique : i+1², i+2², ...
  Double hachage : h1(k) + j*h2(k)
  Python dict (depuis 3.6), C++ unordered_map
```

## Facteur de charge et rehashing
```
load_factor = n_elements / capacité_tableau

Si load_factor > seuil (typiquement 0.75) :
  → rehashing : doubler la capacité, réinsérer tous les éléments
  → O(n) ponctuel, O(1) amorti sur la durée

Python dict : seuil ≈ 2/3
Java HashMap : seuil = 0.75 par défaut
```

> [!tip] set pour les tests d'appartenance répétés
> `x in my_list` : O(n) — lent pour de grandes listes
> `x in my_set`  : O(1) — indépendant de la taille
> Convertir en set avant de faire n recherches : O(n) une fois, puis O(1) par recherche.
