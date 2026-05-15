#algorithmique #structures-de-données #glossaire #référence

| Terme | Définition |
|---|---|
| **LIFO** | Last In First Out — pile |
| **FIFO** | First In First Out — file |
| **pile (stack)** | Structure LIFO — push/pop au sommet en O(1) |
| **file (queue)** | Structure FIFO — enqueue en queue, dequeue en tête en O(1) |
| **deque** | Double-ended queue — insertions/suppressions O(1) des deux côtés |
| **file de priorité** | File où dequeue() retourne l'élément de priorité min (ou max) |
| **tas (heap)** | Arbre binaire presque complet respectant la propriété de tas |
| **min-heap** | Tas où chaque parent ≤ ses enfants — racine = minimum |
| **max-heap** | Tas où chaque parent ≥ ses enfants — racine = maximum |
| **sift-up** | Remonter un nœud vers le haut du tas après insertion |
| **sift-down** | Descendre un nœud vers le bas du tas après extraction |
| **heapify** | Construire un tas depuis un tableau en O(n) |
| **heap sort** | Tri par tas — O(n log n), en place, non stable |
| **liste chaînée** | Nœuds liés par des pointeurs — insertion O(1) tête, accès O(n) |
| **liste doublement chaînée** | Liste chaînée avec pointeurs dans les deux sens |
| **nœud sentinelle** | Nœud fictif en tête ou queue pour simplifier les opérations sur les bords |
| **algorithme de Floyd** | Détection de cycle dans une liste chaînée avec deux pointeurs (lent/rapide) |
| **ABR (BST)** | Arbre Binaire de Recherche — gauche < racine < droite |
| **arbre équilibré** | ABR dont la hauteur est garantie O(log n) — AVL, rouge-noir |
| **arbre dégénéré** | ABR dont la hauteur est O(n) — insertions en ordre croissant |
| **inorder** | Parcours ABR : gauche → racine → droite → valeurs triées |
| **preorder** | Parcours : racine → gauche → droite |
| **postorder** | Parcours : gauche → droite → racine |
| **trie** | Arbre de préfixes — partage les préfixes communs des chaînes |
| **table de hachage** | Structure clé→valeur avec accès O(1) moyen via hash |
| **collision** | Deux clés différentes → même index dans le tableau de hachage |
| **chaînage** | Gestion des collisions par liste à chaque case |
| **adressage ouvert** | Gestion des collisions en cherchant la prochaine case libre |
| **facteur de charge** | n_éléments / capacité — déclenche le rehashing quand trop élevé |
| **rehashing** | Redimensionner la table et réinsérer tous les éléments — O(n) ponctuel |
| **hashable** | Objet pouvant être clé de hash map — doit implémenter hash + égalité |
| **Union-Find (DSU)** | Structure pour ensembles disjoints avec find/union quasi-O(1) |
| **composante connexe** | Ensemble de nœuds reliés dans un graphe |
| **union par rang** | Optimisation Union-Find — attacher le petit arbre sous le grand |
| **compression de chemin** | Optimisation Union-Find — aplatir vers la racine à chaque find |
| **α (alpha)** | Fonction inverse d'Ackermann — complexité Union-Find optimisé ≈ O(1) |
| **MST** | Minimum Spanning Tree — arbre couvrant minimal d'un graphe pondéré |
| **lazy deletion** | Marquer un élément comme supprimé sans le retirer physiquement du tas |
| **decrease-key** | Modifier la priorité d'un élément dans un tas — non supporté par les implémentations standard |
| **LRU cache** | Least Recently Used — cache qui évince l'élément le moins récemment utilisé |
| **complexité amortie** | Coût moyen sur n opérations — ex: O(1) amorti pour l'insertion tableau dynamique |
| **Big-O** | Borne supérieure asymptotique — comportement dans le pire cas |
