#algo #graphes #dfs #parcours

## DFS — Depth-First Search

S'enfonce le plus loin possible dans une branche avant de revenir. Utilise une **pile LIFO** — explicite ou via la récursion.

## Implémentation récursive

```python
def dfs(graph, node, visited=None):
    if visited is None:
        visited = set()

    visited.add(node)
    print(node)                          # traitement du nœud

    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited) # ✅ récursion = pile implicite

    return visited
```

## Implémentation itérative (pile explicite)

```python
def dfs_iter(graph, start):
    visited = set()
    stack   = [start]
    order   = []

    while stack:
        node = stack.pop()               # ✅ retire du SOMMET (LIFO)
        if node in visited:
            continue
        visited.add(node)
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)

    return order
```

## DFS avec timestamps (pré/post-ordre)

```python
def dfs_timestamps(graph, start):
    visited, timer = {}, [0]

    def dfs(node):
        timer[0] += 1
        visited[node] = {'pre': timer[0], 'post': None}
        for nb in graph[node]:
            if nb not in visited:
                dfs(nb)
        timer[0] += 1
        visited[node]['post'] = timer[0]  # ✅ utilisé pour le tri topologique

    dfs(start)
    return visited
```

## Illustration

```
Graphe :     DFS depuis A (ordre récursif) :
A — B — E    A → B → E (dead end) ↩ → D ↩ ↩ → C
|   |
C — D

Pile à chaque étape :
[A] → pop A → push C,B
[C,B] → pop B → push D,E
[C,D,E] → pop E → ...
```

## Complexité

| | Valeur |
|--|--------|
| Temps | O(V + E) |
| Espace | O(V) — pile de récursion ou pile explicite |

## Usages principaux

| Problème | Pourquoi DFS |
|----------|-------------|
| Détection de cycle | Arc arrière = cycle |
| Tri topologique | Ordre des timestamps post |
| Composantes fortement connexes | Algorithme de Tarjan/Kosaraju |
| Backtracking (labyrinthes, N-reines) | Exploration exhaustive naturelle |

> [!warning] Stack overflow
> La version récursive risque un dépassement de pile sur les graphes très profonds (V > ~1000 en Python). Préférer la version itérative dans ce cas.

> [!tip] DFS vs BFS — la seule différence de code
> Remplacer `queue.popleft()` par `stack.pop()`. Un mot change, le comportement est radicalement différent.
