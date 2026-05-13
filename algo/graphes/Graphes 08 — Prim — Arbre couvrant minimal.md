#algo #graphes #prim #MST

## Prim — Arbre couvrant minimal

Construit le MST en **faisant grandir un arbre** depuis un nœud de départ, en ajoutant toujours l'arête la moins chère qui connecte un nouveau nœud. Pense en **voisinage local**.

## Propriétés

- Produit le même MST que Kruskal (résultat identique, stratégie différente).
- Utilise une **file de priorité** (tas min) sur les arêtes candidates.
- Optimal pour les **graphes denses** (beaucoup d'arêtes).
- Complexité avec tas binaire : O(E log V).

## Implémentation

```python
import heapq

def prim(graph, start):
    in_tree = {start}
    # (poids, nœud_source, nœud_destination)
    heap    = [(w, start, nb) for nb, w in graph[start]]
    heapq.heapify(heap)
    mst, cost = [], 0

    while heap and len(in_tree) < len(graph):
        w, src, dst = heapq.heappop(heap)

        if dst in in_tree:
            continue                          # ✅ déjà dans l'arbre, ignorer

        in_tree.add(dst)
        mst.append((src, dst, w))
        cost += w

        for neighbor, nw in graph[dst]:       # ✅ ajouter les nouveaux candidats
            if neighbor not in in_tree:
                heapq.heappush(heap, (nw, dst, neighbor))

    return mst, cost
```

## Illustration — croissance de l'arbre

```
Graphe (départ depuis A) :

A──4──B        Étape 1 : A dans l'arbre. Candidats : A–B:4, A–C:3
|    |         Étape 2 : pop A–C:3 ✅ → C rejoint. Candidats: A–B:4, C–D:5
3    2         Étape 3 : pop A–B:4 ✅ → B rejoint. Candidats: B–C:1❌(déjà in_tree), B–D:2
|    |         Étape 4 : pop B–D:2 ✅ → D rejoint.
C──1──D

MST : A–C:3, A–B:4, B–D:2   coût = 9
```

## Complexité

| Structure | Temps |
|-----------|-------|
| Tableau simple | O(V²) |
| Tas binaire (`heapq`) | O(E log V) |
| Tas de Fibonacci | O(E + V log V) |

## Kruskal vs Prim

| Critère | Kruskal | Prim |
|---------|---------|------|
| Approche | Arêtes globales (tri) | Croissance locale |
| Structure | Union-Find | File de priorité |
| Graphe épars (E ≈ V) | ✅ | ⚠️ |
| Graphe dense (E ≈ V²) | ⚠️ | ✅ |
| Implémentation | Nécessite Union-Find | Plus directe |
| Résultat | Même MST optimal | Même MST optimal |

> [!tip] Prim = Dijkstra sans accumuler les poids
> Prim choisit l'arête de poids minimum vers un nouveau nœud. Dijkstra choisit le nœud de distance cumulée minimum. Même structure de code, sémantique légèrement différente.

> [!warning] Graphes non connexes
> Sur un graphe non connexe, Prim ne couvre qu'une composante. Pour couvrir tout le graphe, relancer depuis un nœud non visité de chaque composante → forêt couvrante minimale.
