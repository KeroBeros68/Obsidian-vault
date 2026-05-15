#algorithmique #structures-de-données #complexité #big-o #référence

## Notation Big-O — rappel
| Notation | Nom | Exemple concret |
|---|---|---|
| O(1) | Constant | Accès dict, push pile, peek heap |
| O(log n) | Logarithmique | Recherche binaire, ABR équilibré, heap push/pop |
| O(n) | Linéaire | Parcours, recherche liste, heapify |
| O(n log n) | Quasi-linéaire | Tri fusion, Dijkstra, heap sort |
| O(n²) | Quadratique | Tri bulle, Floyd-Warshall dense |
| O(2ⁿ) | Exponentiel | Énumération de sous-ensembles |

## Tableau de synthèse — toutes les structures
| Structure | Accès idx | Recherche | Insert | Suppression | Espace |
|---|---|---|---|---|---|
| **Tableau / list** | O(1) | O(n) | O(1)* fin / O(n) | O(1) fin / O(n) | O(n) |
| **Deque** | O(n) | O(n) | O(1) tête/queue | O(1) tête/queue | O(n) |
| **Pile (stack)** | — | O(n) | O(1)* sommet | O(1) sommet | O(n) |
| **File (queue)** | — | O(n) | O(1) queue | O(1) tête | O(n) |
| **File de priorité** | O(1) min | O(n) | O(log n) | O(log n) min | O(n) |
| **Liste chaînée** | O(n) | O(n) | O(1) tête | O(1) sur ptr | O(n) |
| **Hash Map / Set** | O(1)† | O(1)† | O(1)† | O(1)† | O(n) |
| **ABR équilibré** | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| **Tas binaire** | O(1) min | O(n) | O(log n) | O(log n) min | O(n) |
| **Trie** | O(m) | O(m) | O(m) | O(m) | O(n·m·Σ) |
| **Union-Find** | — | O(α(n)) | O(α(n)) | — | O(n) |

`*` amorti — `†` O(1) moyen, O(n) pire cas — `m` longueur du mot/clé — `Σ` taille alphabet — `α` ≈ O(1)

## Quel outil pour quel besoin ?
| Besoin | Structure | Langages |
|---|---|---|
| Pile LIFO | `list` / `deque` / `stack` | Python list, Java ArrayDeque, C++ stack |
| File FIFO | `deque` / `Queue` | Python deque, Java ArrayDeque, C++ queue |
| File thread-safe | `BlockingQueue` / `queue.Queue` | Java LinkedBlockingQueue, Python queue.Queue |
| File de priorité | `heapq` / `PriorityQueue` | Python heapq, Java PriorityQueue, C++ priority_queue |
| Dictionnaire | `dict` / `HashMap` | Python dict, Java HashMap, C++ unordered_map |
| Ensemble unique | `set` / `HashSet` | Python set, Java HashSet, C++ unordered_set |
| Dict ordonné | `OrderedDict` / `LinkedHashMap` | Python OrderedDict, Java LinkedHashMap |
| Dict trié | `SortedDict` / `TreeMap` | Python sortedcontainers, Java TreeMap, C++ map |
| Mots avec préfixes | `Trie` | Implémenter |
| Composantes connexes | `UnionFind` | Implémenter |
| Cache LRU | Liste chaînée + Hash Map | Python lru_cache, Java LinkedHashMap |

## Complexités des algorithmes de graphes — dépendance aux structures
| Algorithme | Structure clé | Complexité | Fiche |
|---|---|---|---|
| BFS | File FIFO (deque) | O(V + E) | [[Graphes 02 — BFS — Parcours en largeur]] |
| DFS | Pile (récursion/stack) | O(V + E) | [[Graphes 03 — DFS — Parcours en profondeur]] |
| Dijkstra | File de priorité (heap) | O((V+E) log V) | [[Graphes 05 — Dijkstra — Plus court chemin]] |
| A* | File de priorité (heap) | O(E log V) | [[Graphes 06 — A-star — Recherche heuristique]] |
| Prim | File de priorité (heap) | O((V+E) log V) | [[Graphes 08 — Prim — Arbre couvrant minimal]] |
| Kruskal | Union-Find | O(E log E) | [[Graphes 07 — Kruskal — Arbre couvrant minimal]] |

## Croissance des complexités — valeurs concrètes
| n | O(log n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) |
|---|---|---|---|---|---|
| 10 | 3 | 10 | 33 | 100 | 1 024 |
| 100 | 7 | 100 | 664 | 10 000 | 10³⁰ |
| 1 000 | 10 | 1 000 | 10 000 | 10⁶ | ∞ |
| 10 000 | 13 | 10 000 | 130 000 | 10⁸ | ∞ |
| 10⁶ | 20 | 10⁶ | 2×10⁷ | 10¹² | ∞ |

## Résumé des structures natives par langage
| Structure | Python | Java | C++ | JavaScript | Go |
|---|---|---|---|---|---|
| Tableau dynamique | `list` | `ArrayList` | `vector` | `Array` | `slice` |
| Deque | `deque` | `ArrayDeque` | `deque` | *(manuel)* | `list.List` |
| Hash Map | `dict` | `HashMap` | `unordered_map` | `Map` | `map` |
| Hash Set | `set` | `HashSet` | `unordered_set` | `Set` | `map[T]bool` |
| Dict trié | `SortedDict`* | `TreeMap` | `map` | *(manuel)* | *(manuel)* |
| File de priorité | `heapq` | `PriorityQueue` | `priority_queue` | *(manuel)* | `container/heap` |
| Pile | `list` | `ArrayDeque` | `stack` | `Array` | `slice` |
| File FIFO | `deque` | `ArrayDeque` | `queue` | *(manuel)* | channel / `list.List` |

`*` = bibliothèque externe `sortedcontainers`
