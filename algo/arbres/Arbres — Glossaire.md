#algo #arbres #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **nœud (node)** | Élément de l'arbre contenant une valeur et des pointeurs vers ses enfants. |
| **racine (root)** | Nœud unique sans parent — sommet de l'arbre. |
| **feuille (leaf)** | Nœud sans enfant. |
| **hauteur (h)** | Longueur du chemin le plus long racine → feuille. Convention courante : arbre vide = −1, feuille = 0. |
| **profondeur** | Distance d'un nœud à la racine. La racine a profondeur 0. |
| **facteur d'équilibre (bf)** | `h(gauche) − h(droite)`. Invariant AVL : `bf ∈ {−1, 0, +1}`. |
| **arbre complet** | Tous les niveaux remplis sauf éventuellement le dernier (rempli de gauche). Structure du tas binaire. |
| **arbre parfait** | Tous les niveaux entièrement remplis. Nombre de nœuds = 2^(h+1) − 1. |
| **arbre dégénéré** | Hauteur O(n) — chaque nœud a au plus un enfant. Équivalent à une liste chaînée. |
| **successeur infixe** | Plus petit nœud du sous-arbre droit d'un nœud N. Utilisé dans la suppression BST cas 3. |
| **prédécesseur infixe** | Plus grand nœud du sous-arbre gauche. Alternative au successeur pour la suppression BST. |
| **rotation simple** | Réorganisation locale (LL ou RR) qui restaure le facteur d'équilibre AVL. |
| **rotation double** | Deux rotations simples consécutives (LR ou RL). |
| **black-height** | Nombre de nœuds NOIRS sur tout chemin racine → feuille (NIL) dans un arbre rouge-noir. Invariant 5. |
| **sentinelle NIL** | Feuille fictive (nœud noir) dans un rouge-noir. Simplifie les cas limites de l'algorithme. |
| **B-Tree d'ordre m** | Arbre équilibré multi-voies : chaque nœud a entre ⌈m/2⌉−1 et m−1 clés et entre ⌈m/2⌉ et m enfants. |
| **degré minimum t** | Convention CLRS pour le B-Tree : chaque nœud a entre t−1 et 2t−1 clés (lien : t = ⌈m/2⌉). |
| **split (B-Tree)** | Division d'un nœud plein (2t−1 clés) en deux nœuds (t−1 clés chacun), la clé médiane remontant au parent. |
| **merge (B-Tree)** | Fusion d'un nœud sous-plein avec un frère + descente d'une clé du parent. Inverse du split. |
| **B+ Tree** | Variante du B-Tree : données uniquement dans les feuilles, feuilles reliées en liste. Utilisé par PostgreSQL/MySQL. |
| **in-order (infixe)** | Gauche → racine → droite. Produit les valeurs triées sur un BST. |
| **pre-order (préfixe)** | Racine → gauche → droite. Sérialisation, copie fidèle de la structure. |
| **post-order (suffixe)** | Gauche → droite → racine. Suppression sûre, évaluation d'AST. |
| **level-order (BFS)** | Parcours niveau par niveau via une file FIFO. Voir [[Graphes 02 — BFS — Parcours en largeur]]. |
| **sift-up / heapify-up** | Remontée d'un élément vers la racine après insertion dans le tas. |
| **sift-down / heapify-down** | Descente d'un élément depuis la racine après extraction du minimum. |
