#algo #graphes #kruskal #MST #union-find

## Kruskal — Arbre couvrant minimal

Construit le MST en triant **toutes les arêtes** par poids croissant et en les ajoutant une à une si elles ne créent pas de cycle. Pense en **arêtes globales**.

## Propriétés

- Produit un **arbre couvrant minimal** (MST) : connecte tous les V sommets avec V−1 arêtes et un coût total minimum.
- Utilise la structure **Union-Find** pour détecter les cycles en quasi-O(1).
- Optimal pour les **graphes épars** (peu d'arêtes).
- Complexité dominée par le tri : O(E log E).

## Union-Find — structure clé

```python
def make_uf(nodes):
    return {n: n for n in nodes}, {n: 0 for n in nodes}  # parent, rank

def find(parent, x):
    if parent[x] != x:
        parent[x] = find(parent, parent[x])  # ✅ path compression
    return parent[x]

def union(parent, rank, a, b):
    ra, rb = find(parent, a), find(parent, b)
    if ra == rb:
        return False                          # ❌ déjà connectés → cycle
    if rank[ra] < rank[rb]:
        parent[ra] = rb
    elif rank[ra] > rank[rb]:
        parent[rb] = ra
    else:
        parent[rb] = ra
        rank[ra] += 1
    return True                               # ✅ fusionné
```

## Algorithme de Kruskal

```python
def kruskal(nodes, edges):
    edges.sort(key=lambda e: e[2])            # ✅ tri par poids croissant
    parent, rank = make_uf(nodes)
    mst, cost = [], 0

    for a, b, w in edges:
        if union(parent, rank, a, b):         # ✅ pas de cycle → on prend
            mst.append((a, b, w))
            cost += w
            if len(mst) == len(nodes) - 1:
                break                         # ✅ MST complet (V-1 arêtes)

    return mst, cost
```

## Illustration

```
Arêtes triées :  B–C:1, B–D:2, A–B:4, A–C:3, C–D:5...

Étape 1 : B–C:1 → union(B,C)  ✅ acceptée
Étape 2 : B–D:2 → union(B,D)  ✅ acceptée
Étape 3 : A–C:3 → union(A,C)  ✅ acceptée  ← A rejoint le groupe B,C,D
Étape 4 : A–B:4 → find(A)=find(B) ❌ rejetée (cycle !)
...

MST : B–C, B–D, A–C  |  coût = 1+2+3 = 6
```

## Complexité

| Opération | Coût |
|-----------|------|
| Tri des arêtes | O(E log E) |
| Union-Find (path compression + rank) | O(α(V)) ≈ O(1) par opération |
| Total | **O(E log E)** |

> [!tip] Graphe épars → Kruskal
> Si E ≈ V (peu d'arêtes), le tri est rapide. Kruskal est le choix naturel pour les graphes épars comme les réseaux de routes.

> [!info] Union-Find avec path compression
> `find()` avec compression de chemin aplatit l'arbre à chaque appel. L'union par rang évite les arbres dégénérés. Ensemble, ils donnent O(α(n)) ≈ O(1) amorti — quasi-constant pour toutes valeurs pratiques.
