#algo #graphes #dijkstra #plus-court-chemin

## Dijkstra

Plus court chemin depuis une source vers tous les sommets d'un graphe **pondéré à poids positifs**. Explore en ordre croissant de distance cumulée.

## Propriétés

- Fonctionne uniquement avec des **poids ≥ 0** (utiliser Bellman-Ford sinon).
- Garantit le chemin optimal à chaque sommet finalisé.
- Avec un tas binaire : O((V + E) log V).
- Avec un tas de Fibonacci : O(E + V log V) — optimal sur graphes denses.

## Implémentation

```python
import heapq

def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    prev = {node: None for node in graph}
    heap = [(0, start)]                  # (distance, nœud)

    while heap:
        d, node = heapq.heappop(heap)

        if d > dist[node]:
            continue                     # ✅ nœud déjà finalisé, ignorer

        for neighbor, weight in graph[node]:
            new_dist = dist[node] + weight
            if new_dist < dist[neighbor]:
                dist[neighbor] = new_dist
                prev[neighbor] = node
                heapq.heappush(heap, (new_dist, neighbor))

    return dist, prev

def reconstruct(prev, target):
    path = []
    while target is not None:
        path.append(target)
        target = prev[target]
    return path[::-1]                    # ✅ inverser pour avoir source→cible
```

## Illustration

```
Graphe pondéré :        État de la file (dist, nœud) :
S -4→ A                 Init   : [(0,S)]
S -2→ B                 Pop S  : [(2,B),(4,A)]
A -5→ C                 Pop B  : [(3,D),(4,A)]  ← B+1=3 pour D
B -1→ A                 Pop D  : [(4,A),(7,F)]
B -3→ D                 Pop A  : [(7,C),(7,F)]  ← A+5=7 pour C... etc.
...

dist final : S=0, B=2, A=3, D=3, C=8, T=9
```

## Complexité

| Structure | Temps |
|-----------|-------|
| Tableau simple | O(V²) |
| Tas binaire (`heapq`) | O((V + E) log V) |
| Tas de Fibonacci | O(E + V log V) |

> [!tip] Dijkstra = BFS pondéré
> Dijkstra est conceptuellement un BFS où la file FIFO est remplacée par une file de priorité. La "distance" joue le rôle des "couches" de BFS.

> [!warning] Poids négatifs
> Un poids négatif casse Dijkstra — un nœud finalisé pourrait être amélioré plus tard. Utiliser Bellman-Ford (O(VE)) pour les graphes avec arcs négatifs.

> [!example] Graphe d'entrée attendu
> ```python
> graph = {
>     'S': [('A', 4), ('B', 2)],
>     'A': [('C', 5)],
>     'B': [('A', 1), ('D', 3)],
>     # ...
> }
> ```
