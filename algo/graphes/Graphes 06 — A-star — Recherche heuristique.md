#algo #graphes #astar #heuristique #plus-court-chemin

## A* — Recherche heuristique

Extension de Dijkstra guidée par une **heuristique h(n)** estimant la distance restante jusqu'à la destination. Explore moins de nœuds que Dijkstra en orientant la recherche.

## Propriétés

- **Optimal** si h(n) est **admissible** : h(n) ≤ distance réelle restante (jamais surestimée).
- **Complet** : trouve toujours une solution si elle existe.
- Si h(n) = 0 pour tout n → comportement identique à Dijkstra.
- Si h(n) surestime → plus rapide mais pas forcément optimal.

## Formule centrale

```
f(n) = g(n) + h(n)

g(n) = coût réel depuis la source jusqu'à n (connu)
h(n) = estimation du coût restant de n jusqu'à la cible (heuristique)
f(n) = priorité dans la file — le plus petit passe en premier
```

## Implémentation

```python
import heapq

def astar(graph, start, goal, heuristic):
    open_set = [(heuristic(start), 0, start)]  # (f, g, nœud)
    g_score  = {start: 0}
    prev     = {start: None}
    closed   = set()

    while open_set:
        f, g, node = heapq.heappop(open_set)

        if node == goal:
            return reconstruct(prev, goal)      # ✅ chemin trouvé

        if node in closed:
            continue
        closed.add(node)

        for neighbor, weight in graph[node]:
            new_g = g + weight
            if new_g < g_score.get(neighbor, float('inf')):
                g_score[neighbor] = new_g
                prev[neighbor]    = node
                f_score           = new_g + heuristic(neighbor)
                heapq.heappush(open_set, (f_score, new_g, neighbor))

    return None                                 # aucun chemin

def reconstruct(prev, node):
    path = []
    while node is not None:
        path.append(node)
        node = prev[node]
    return path[::-1]
```

## Heuristiques courantes

| Contexte | Heuristique | Admissible ? |
|----------|-------------|-------------|
| Grille (dépl. 4 dir.) | Distance de Manhattan | ✅ |
| Grille (dépl. 8 dir.) | Distance de Chebyshev | ✅ |
| Espace euclidien | Distance à vol d'oiseau | ✅ |
| Graphe abstrait | 0 (→ Dijkstra) | ✅ |
| Jeux vidéo (approx.) | Heuristique pondérée ε·h | ⚠️ non admissible |

```python
# Exemples de heuristiques sur grille 2D
def manhattan(a, b):
    return abs(a[0]-b[0]) + abs(a[1]-b[1])     # ✅ 4 directions

def euclidean(a, b):
    return ((a[0]-b[0])**2 + (a[1]-b[1])**2)**0.5  # ✅ toutes directions
```

## Illustration

```
A* depuis S vers T :
Dijkstra explore toutes les directions.
A* oriente la recherche vers T via h(n).

     h=6  h=4  h=2
S ──→ A ──→ C ──→ T
 \       \
  B──→D   h = distance à vol d'oiseau vers T
 h=5  h=4

A* finalise ~40% moins de nœuds que Dijkstra sur ce graphe.
```

> [!tip] Dijkstra est un cas particulier de A*
> A* avec h(n)=0 est exactement Dijkstra. A* est le cadre général, Dijkstra est le cas sans information de guidage.

> [!warning] Admissibilité de l'heuristique
> Une heuristique non admissible (qui surestime) peut rater le chemin optimal. Toujours vérifier que h(n) ≤ distance_réelle(n, cible) pour garantir l'optimalité.
