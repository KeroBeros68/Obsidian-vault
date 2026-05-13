#algo #graphes #bfs #parcours

## BFS — Breadth-First Search

Explore le graphe couche par couche, en visitant tous les voisins directs avant d'aller plus loin. Utilise une **file FIFO**.

## Implémentation

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue   = deque([start])
    order   = []

    while queue:
        node = queue.popleft()          # ✅ retire du DEVANT (FIFO)
        order.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)  # ✅ ajoute en QUEUE

    return order
```

## Plus court chemin (non pondéré)

```python
def bfs_shortest(graph, start, target):
    visited = {start}
    queue   = deque([(start, [start])])  # (nœud, chemin)

    while queue:
        node, path = queue.popleft()
        if node == target:
            return path                  # ✅ chemin optimal garanti
        for nb in graph[node]:
            if nb not in visited:
                visited.add(nb)
                queue.append((nb, path + [nb]))
    return None                          # aucun chemin
```

## Illustration — exploration par couches

```
Graphe :          Ordre BFS depuis A :
A — B — E         Couche 0 : A
|   |             Couche 1 : B, C
C — D             Couche 2 : E, D

A → B,C → E,D
```

## Complexité

| | Valeur |
|--|--------|
| Temps | O(V + E) |
| Espace | O(V) — file + ensemble visités |

> [!tip] BFS = plus court chemin
> Sur un graphe non pondéré, BFS garantit le chemin le plus court en nombre d'arêtes. C'est sa propriété clé.

> [!warning] Graphes pondérés
> BFS ne donne pas le plus court chemin si les arêtes ont des poids différents. Utiliser Dijkstra à la place.

> [!info] Variantes
> BFS bidirectionnel (depuis source ET destination) réduit la complexité en pratique. Utilisé dans les GPS et les réseaux sociaux (degrés de séparation).
