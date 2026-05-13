#algo #graphes #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Graphe G=(V,E)** | Ensemble de sommets V reliés par des arêtes E. |
| **Sommet / Nœud** | Entité du graphe. Noté u, v, ou par un label. |
| **Arête** | Lien entre deux sommets. Non orientée : (u,v) = (v,u). |
| **Arc** | Lien orienté u→v dans un digraphe. u→v ≠ v→u. |
| **Degré** | Nombre d'arêtes incidentes à un sommet. Noté deg(v). |
| **Degré entrant (in-degree)** | Nombre d'arcs arrivant sur v dans un digraphe. |
| **Degré sortant (out-degree)** | Nombre d'arcs partant de v dans un digraphe. |
| **Chemin** | Suite de sommets u₀,u₁,...,uₖ où chaque (uᵢ,uᵢ₊₁) est une arête. |
| **Cycle** | Chemin dont le premier et dernier sommet sont identiques. |
| **Graphe connexe** | Il existe un chemin entre toute paire de sommets. |
| **Composante connexe** | Sous-graphe maximal connexe. |
| **DAG** | Directed Acyclic Graph — graphe orienté sans cycle. |
| **Arbre** | Graphe connexe sans cycle. Propriété : E = V−1. |
| **Arbre couvrant** | Sous-graphe couvrant tous les sommets, formant un arbre. |
| **MST** | Minimum Spanning Tree — arbre couvrant de poids total minimal. |
| **Arbre couvrant minimal** | Synonyme français de MST. |
| **BFS** | Breadth-First Search — parcours en largeur, couche par couche. |
| **DFS** | Depth-First Search — parcours en profondeur, branche par branche. |
| **File de priorité** | Structure où l'élément de priorité minimale (ou max) est extrait en O(log n). |
| **Union-Find (DSU)** | Disjoint Set Union — structure pour gérer des partitions, fusionner des composantes en quasi-O(1). |
| **Path compression** | Optimisation d'Union-Find : aplatir l'arbre lors de `find` pour accélérer les appels futurs. |
| **Union by rank** | Optimisation d'Union-Find : attacher le petit arbre sous le grand pour éviter les dégénérescences. |
| **Heuristique** | Fonction h(n) estimant le coût restant dans A*. Admissible si h(n) ≤ coût_réel. |
| **Relaxation** | Mise à jour de dist[v] si un chemin plus court via u est trouvé : dist[u] + w(u,v) < dist[v]. |
| **Tri topologique** | Ordre linéaire des sommets d'un DAG respectant toutes les dépendances. |
| **Tas binaire** | Arbre binaire complet respectant la propriété de tas. Insert/Extract en O(log n). |
| **Tas de Fibonacci** | Forêt de tas paresseuse. Decrease-key en O(1) amorti. |
| **Analyse amortie** | Technique d'analyse distribuant le coût des opérations coûteuses sur les opérations précédentes. |
| **Potentiel Φ** | Fonction mesurant l'"énergie" stockée dans une structure pour l'analyse amortie. |
| **Consolidation** | Étape d'extract-min du tas Fibonacci : fusionner les arbres de même degré. |
| **Coupure en cascade** | Dans le tas Fibonacci, propagation des coupures vers les ancêtres marqués lors d'un decrease-key. |
