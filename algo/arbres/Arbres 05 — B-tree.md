#algo #arbres #b-tree #base-de-données #index #avancé

## Définition (ordre m / degré minimum t)

```
Convention CLRS (degré minimum t) :
  Chaque nœud non-racine contient entre t−1 et 2t−1 clés.
  Chaque nœud interne a entre t et 2t enfants.
  Toutes les feuilles sont à la même profondeur.
  Les clés dans un nœud sont triées.

Lien avec l'ordre m : m = 2t (chaque nœud est rempli au moins à moitié)
```

Hauteur h = O(log_t n) — très faible pour t grand.
Exemple : t = 500 → h ≤ 4 pour n = 10⁹ lignes.

## Structure — exemple t = 2 (2-3-4 tree)

```
           [17 | 35]
          /    |     \
    [5|11]  [20|28]  [40|50]
    / | \    / | \    / | \
  [1][7][12][18][22][30][38][43]
```

Chaque nœud interne : 2 à 4 enfants, 1 à 3 clés.

## Insertion avec split

```
Algorithme proactif (CLRS) :
1. Descendre vers la feuille cible.
2. À chaque nœud plein (2t−1 clés) rencontré à la descente → SPLIT immédiat :
   a. Diviser le nœud en deux moitiés (t−1 clés chacune).
   b. Remonter la clé médiane au parent.
   c. Le parent n'est jamais plein (splité à la descente) → pas de cascade.
3. Insérer dans la feuille (garantie non-pleine).
```

```
Exemple — split du nœud [1|3|5|7] (t=2, plein à 3 clés) :
  [1|3|5|7]  →  [1]  médiane=3  [5|7]
  La clé 3 remonte au parent.
```

## Suppression avec merge / redistribution

```
1. Localiser la clé cible (descente BST multi-voies).
2. Si clé dans un nœud interne → remplacer par le successeur infixe (feuille).
3. Après suppression dans la feuille, si le nœud passe sous t−1 clés :
   a. Redistribuer : emprunter une clé à un frère adjacent (via le parent).
   b. Fusionner : si redistribution impossible → merger avec un frère + descendre
      une clé du parent → si le parent passe sous t−1 → remonter le merge.
```

## Variantes

| Variante | Différence clé | Usage typique |
|---|---|---|
| **B+ Tree** | Données uniquement dans les feuilles, feuilles liées en liste chaînée | PostgreSQL, MySQL InnoDB, SQLite |
| **B* Tree** | Nœuds remplis au moins aux ⅔ avant split | Systèmes de fichiers denses |
| **B Tree** | Données dans tous les nœuds | HFS+, systèmes embarqués |

## Pourquoi les bases de données utilisent le B+ Tree

```
1. Feuilles liées → range query O(log n + k) sans remonter à la racine.
2. Données uniquement en feuilles → scan séquentiel complet sans traversée interne.
3. Nœuds larges (t = 100-500) → 1 nœud = 1 page disque (4 ou 8 Ko).
   → Lectures disque minimisées : h ≤ 3-4 niveaux pour des milliards de lignes.
```

## Complexités

| Opération | B-Tree (t minimal, n clés) |
|---|---|
| search | O(log_t n) = O(log n / log t) |
| insert | O(log_t n) — amorti |
| delete | O(log_t n) — amorti |
| Range query (k résultats) | O(log_t n + k) — B+ Tree uniquement |
| Hauteur | O(log_t n) |

> [!tip] Intuition disque
> Chaque accès à un nœud = 1 I/O disque.
> Maximiser t réduit la hauteur et donc les I/O.
> C'est pourquoi t = 100-500 en production (une page disque = un nœud B-tree).

> [!warning] B-Tree ≠ Binary Tree
> Un B-Tree est multi-voies (jusqu'à 2t enfants par nœud).
> Confondre les deux change radicalement l'analyse de complexité (log_t vs log₂).
