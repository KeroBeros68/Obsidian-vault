#algo #graphes #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier de marquer un nœud avant de l'enfiler

```python
# ❌ Marquer à la sortie → visite les nœuds plusieurs fois
while queue:
    node = queue.popleft()
    visited.add(node)           # trop tard
    for nb in graph[node]:
        if nb not in visited:
            queue.append(nb)    # nb peut être ajouté plusieurs fois

# ✅ Marquer à l'entrée dans la file
visited = {start}
queue = deque([start])
while queue:
    node = queue.popleft()
    for nb in graph[node]:
        if nb not in visited:
            visited.add(nb)     # ✅ marquer immédiatement
            queue.append(nb)
```

> [!warning] Doublons en file
> Sur un graphe dense, un nœud peut être enfilé O(E) fois si on ne le marque qu'à la sortie. Cela transforme un O(V+E) en O(E²) dans le pire cas.

---

## 🪤 Piège 2 — Ignorer les entrées obsolètes dans la file de Dijkstra

```python
# ❌ Ne pas filtrer les entrées périmées
while heap:
    d, node = heapq.heappop(heap)
    for nb, w in graph[node]:   # on retraite des nœuds déjà finalisés

# ✅ Vérifier que la distance est encore valide
while heap:
    d, node = heapq.heappop(heap)
    if d > dist[node]:          # ✅ entrée périmée → ignorer
        continue
    for nb, w in graph[node]:
        ...
```

> [!warning] Correctness vs performance
> Sans ce check, Dijkstra reste correct (il ne met plus à jour `dist` si la valeur est déjà meilleure) mais traite des entrées inutiles. Sur un graphe dense, cela peut multiplier le temps d'exécution.

---

## 🪤 Piège 3 — Appliquer Dijkstra sur un graphe avec poids négatifs

```python
graph = {
    'A': [('B', 4), ('C', -2)],  # ❌ poids négatif
    'C': [('B', 1)],
}
# Dijkstra peut finaliser B avec dist=4 avant de découvrir
# le chemin A→C→B de coût -2+1 = -1. Résultat faux.
```

> [!warning] Poids négatifs → Bellman-Ford
> Utiliser Bellman-Ford O(VE) pour les graphes avec arcs négatifs. Si un cycle négatif existe, Bellman-Ford le détecte explicitement.

---

## 🪤 Piège 4 — Tri topologique sur un graphe avec cycle

```python
# ❌ Kahn silencieux sur un cycle
order = kahn(graph)
# Si len(order) < len(nodes), il y a un cycle — ne pas l'ignorer !

# ✅ Toujours vérifier
order = kahn(graph)
if len(order) != len(nodes):
    raise ValueError("Cycle détecté — tri topologique impossible")
```

> [!tip] Mémo
> Kahn : si la file se vide avant d'avoir traité tous les nœuds, c'est qu'il reste des nœuds dont le in-degree n'est jamais tombé à 0 → cycle.

---

## 🪤 Piège 5 — Heuristique non admissible dans A*

```python
# ❌ Heuristique qui surestime → peut rater le chemin optimal
def h(node):
    return distance_surestimée(node, goal)  # A* plus rapide mais faux

# ✅ Heuristique admissible : toujours ≤ coût réel restant
def h(node):
    return euclidean_distance(node, goal)   # jamais surestimée
```

> [!warning] Admissibilité
> Une heuristique non admissible transforme A* en un algorithme glouton non optimal. Si l'optimalité est requise, vérifier formellement que h(n) ≤ coût_réel pour tout n.

---

## 🪤 Piège 6 — Confusion entre Kruskal et Union-Find sur les graphes orientés

```python
# ❌ Kruskal suppose un graphe NON ORIENTÉ
# Sur un graphe orienté, l'arête A→B ≠ B→A
# Le MST n'est défini que sur les graphes non orientés.
# Pour les digraphes, utiliser l'arborescence couvrant minimum (algorithme de Edmonds).
```

> [!info] MST = graphe non orienté uniquement
> Sur un digraphe, le problème équivalent s'appelle "minimum spanning arborescence" et se résout avec l'algorithme de Chu–Liu/Edmonds, plus complexe.

---

## 🪤 Piège 7 — Stack overflow en DFS récursif

```python
# ❌ Sur un graphe linéaire de 10 000 nœuds
def dfs(node, visited):
    visited.add(node)
    for nb in graph[node]:
        if nb not in visited:
            dfs(nb, visited)  # RecursionError: maximum recursion depth exceeded

# ✅ Version itérative pour les grands graphes
def dfs_iter(graph, start):
    visited, stack = set(), [start]
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            stack.extend(graph[node])
```

> [!tip] Limite Python
> La limite de récursion par défaut en Python est ~1000 (`sys.getrecursionlimit()`). Sur des graphes plus profonds, utiliser la version itérative ou augmenter la limite avec `sys.setrecursionlimit(n)` (avec prudence).

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Nœud marqué trop tard → doublons en file | Marquer à l'enfilage, pas au défilage |
| Entrées périmées dans heap Dijkstra | `if d > dist[node]: continue` |
| Dijkstra avec poids négatifs | Utiliser Bellman-Ford |
| Tri topologique silencieux sur cycle | Vérifier `len(order) == len(nodes)` |
| Heuristique A* non admissible | Garantir h(n) ≤ coût réel restant |
| Kruskal sur digraphe | MST non défini → algorithme de Edmonds |
| DFS récursif → stack overflow | Utiliser la version itérative |
