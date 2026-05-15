#algorithmique #structures-de-données #pièges #erreurs #entretiens

## 🪤 Piège 1 — Utiliser un tableau pour défiler (dequeue) → O(n)
```
Symptôme : file lente sur de grands volumes
Cause    : retirer en tête d'un tableau décale tous les éléments

❌  list.pop(0) / array.shift() / vector.erase(begin())  →  O(n)
✅  deque.popleft() / ArrayDeque.poll() / deque.pop_front() →  O(1)

Règle : si tu retires en tête, utilise une deque, pas un tableau.
```

## 🪤 Piège 2 — Min-heap pris pour un max-heap
```
La plupart des implémentations sont des MIN-heaps par défaut :
  Python heapq, Java PriorityQueue, Go container/heap

heapq.heappop([5,3,8,1])  →  retourne 1 (min), pas 8 (max) !

Pour un max-heap :
  Python  → stocker -valeur, retourner -heappop()
  Java    → new PriorityQueue<>(Collections.reverseOrder())
  C++     → std::priority_queue<int> (max par défaut — exception !)
```

## 🪤 Piège 3 — Tuples non comparables dans un heap
```
Si deux priorités sont égales, Python compare le deuxième élément du tuple.
Si le deuxième élément n'est pas comparable → TypeError ou comportement indéfini.

❌  heappush(pq, (1, mon_objet_non_comparable))

✅  Ajouter un compteur comme tie-breaker :
    from itertools import count
    ctr = count()
    heappush(pq, (priorité, next(ctr), objet))
```

## 🪤 Piège 4 — ABR dégénéré sur données ordonnées
```
Insertions : 1, 2, 3, 4, 5 ...
Résultat   : liste chaînée déguisée — hauteur = n

      1
       \
        2
         \
          3  →  search = O(n), pas O(log n) !

Solution : mélanger les données, ou utiliser un arbre auto-équilibré
  Python → SortedList (sortedcontainers)
  Java   → TreeMap / TreeSet
  C++    → std::map / std::set
```

## 🪤 Piège 5 — Modifier la priorité dans un heap (decrease-key)
```
Les implémentations standard ne supportent pas la modification de priorité.

❌  changer pq[i] directement → propriété de tas violée

✅  Lazy deletion :
    1. Pousser une nouvelle entrée (nouvelle_priorité, élément)
    2. Maintenir un dict des priorités courantes
    3. À l'extraction : ignorer si la priorité ne correspond plus
```

## 🪤 Piège 6 — Union-Find sans les deux optimisations
```
Sans union par rang     → arbre peut atteindre hauteur n → O(n) par find
Sans compression chemin → pas d'amortissement → O(log n) au lieu de O(α(n))

Les deux optimisations sont indépendantes mais complémentaires.
Toujours implémenter les DEUX pour garantir O(α(n)).
```

## 🪤 Piège 7 — Oublier de vérifier connected() avant union() dans Kruskal
```
Si on ne vérifie pas que u et v ne sont pas déjà connectés :
→ On crée un cycle dans le MST → résultat incorrect

✅  Pour chaque arête (u, v, w) triée par poids :
      si NOT connected(u, v) :   ← vérification obligatoire
        union(u, v)
        ajouter au MST
```

## 🪤 Piège 8 — Éléments non hashables comme clé / dans un set
```
Un élément de set ou clé de dict doit être IMMUABLE et hashable.

❌  set.add([1, 2, 3])      →  list n'est pas hashable
❌  dict[[1,2]] = "val"     →  list n'est pas hashable
❌  set.add({1: "a"})       →  dict n'est pas hashable

✅  set.add((1, 2, 3))      →  tuple ✓
✅  set.add(frozenset({1})) →  frozenset ✓

Règle : si tu veux utiliser une collection comme clé → la rendre immuable d'abord.
```

## 🪤 Piège 9 — Modifier un dict/map pendant l'itération
```
❌  for k in my_map:           →  RuntimeError (Python)
      del my_map[k]            →  ConcurrentModificationException (Java)

✅  Itérer sur une copie des clés :
      for k in list(my_map.keys()):
          del my_map[k]

✅  Ou construire un nouveau dict/map sans les clés indésirables
```

## 🪤 Piège 10 — Comparer des valeurs au lieu de références (Floyd)
```
Détecter un cycle ou fusionner des listes :
on compare les RÉFÉRENCES (pointeurs) aux nœuds, pas leurs valeurs.

❌  if slow.val == fast.val → True même si deux nœuds différents ont la même valeur
✅  if slow is fast (Python) / if slow == fast (Java — même objet)
    → comparer les pointeurs, pas les contenus
```

## 🪤 Piège 11 — n insertions au lieu de heapify → O(n log n) vs O(n)
```
❌  pq = []
    for x in data:
        heappush(pq, x)    →  O(n log n)

✅  pq = data[:]
    heapify(pq)            →  O(n)  ← toujours préférer quand les données sont connues à l'avance
```

## 🪤 Piège 12 — Confondre hauteur et profondeur d'un arbre
```
Hauteur   : longueur du plus long chemin d'un nœud vers une FEUILLE
Profondeur: longueur du chemin de la RACINE vers un nœud

ABR équilibré : hauteur ≈ log₂(n)
ABR dégénéré  : hauteur = n - 1
Feuille : hauteur = 0 — arbre vide : hauteur = -1 (convention courante)
```

## Récapitulatif rapide
| Piège | Solution |
|---|---|
| `list.pop(0)` / `shift()` | Utiliser `deque.popleft()` → O(1) |
| Min-heap pris pour max-heap | Inverser les valeurs ou le comparateur |
| Tuples non comparables dans heap | Ajouter un compteur comme tie-breaker |
| ABR dégénéré | Arbre auto-équilibré (`TreeMap`, `SortedList`) |
| Modifier priorité dans heap | Lazy deletion + dict des priorités courantes |
| Union-Find sans optimisations | Toujours rang + compression de chemin |
| Oublier `connected()` dans Kruskal | Vérifier avant chaque `union()` |
| Clé non hashable | `tuple` ou `frozenset` à la place de `list`/`set` |
| Modifier map pendant itération | Itérer sur une copie des clés |
| Comparer valeurs vs références | Comparer les pointeurs (Floyd, fusion) |
| n × heappush au lieu de heapify | `heapify(data)` en O(n) |
