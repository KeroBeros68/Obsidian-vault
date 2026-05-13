#algo #graphes #tri-topologique #DAG

## Tri topologique

Ordre linéaire des sommets d'un **DAG** tel que pour chaque arc u→v, u apparaît avant v. Utilisé pour ordonner des tâches avec dépendances.

## Propriétés

- Applicable uniquement sur un **DAG** (graphe orienté acyclique).
- Un graphe avec cycle n'a pas de tri topologique.
- Le résultat n'est pas unique — plusieurs ordres valides peuvent exister.
- Un graphe à V sommets et E arcs produit un ordre en O(V + E).

## Algorithme de Kahn (BFS)

```python
from collections import deque

def kahn(graph, nodes):
    in_degree = {n: 0 for n in nodes}
    for u in graph:
        for v in graph[u]:
            in_degree[v] += 1           # compter les prérequis

    queue = deque([n for n in nodes if in_degree[n] == 0])  # ✅ sans prérequis
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0: # ✅ prérequis tous traités
                queue.append(neighbor)

    if len(order) != len(nodes):
        raise ValueError("Cycle détecté — tri topologique impossible")  # ❌

    return order
```

## Algorithme DFS (post-ordre inversé)

```python
def topo_dfs(graph, nodes):
    visited, stack = set(), []

    def dfs(node):
        visited.add(node)
        for nb in graph.get(node, []):
            if nb not in visited:
                dfs(nb)
        stack.append(node)              # ✅ empiler à la SORTIE de la récursion

    for node in nodes:
        if node not in visited:
            dfs(node)

    return stack[::-1]                  # ✅ inverser = ordre topologique
```

## Illustration

```
Graphe (dépendances cours) :
Maths ──┐
        ├──→ Algo ──→ Projet ──→ Exam
Cours ──┘              ↑
                       │
               Cours ──┘

Ordre valide : Maths, Cours, Algo, Projet, Exam
In-degrees :   0      0      2     1       1
```

## Détection de cycle

```python
# Kahn : si len(order) < len(nodes) après l'algo → cycle présent ❌
# DFS  : si on revisite un nœud EN COURS DE VISITE → cycle présent ❌

def has_cycle(graph, nodes):
    WHITE, GRAY, BLACK = 0, 1, 2
    color = {n: WHITE for n in nodes}

    def dfs(node):
        color[node] = GRAY
        for nb in graph.get(node, []):
            if color[nb] == GRAY:       # ❌ arc arrière = cycle
                return True
            if color[nb] == WHITE and dfs(nb):
                return True
        color[node] = BLACK
        return False

    return any(dfs(n) for n in nodes if color[n] == WHITE)
```

> [!tip] Kahn vs DFS
> Kahn est plus lisible et détecte les cycles naturellement (vérification de longueur finale). DFS est plus compact si un parcours DFS est déjà présent dans le code.

> [!warning] Unicité
> Un tri topologique unique n'existe que si le graphe est un chemin hamiltonien (chaque nœud a exactement un successeur). En général, plusieurs ordres sont valides.
