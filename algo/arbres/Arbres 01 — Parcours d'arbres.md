#algo #arbres #parcours #traversal

## Les quatre parcours

```
Arbre de référence :
        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4   7 13
```

**Infixe (In-order) — gauche → racine → droite**
Produit les valeurs triées sur un BST.
```
Résultat : 1, 3, 4, 6, 7, 8, 10, 13, 14
```

**Préfixe (Pre-order) — racine → gauche → droite**
Sérialisation, copie fidèle de la structure.
```
Résultat : 8, 3, 1, 6, 4, 7, 10, 14, 13
```

**Suffixe (Post-order) — gauche → droite → racine**
Suppression sûre, évaluation d'expressions ([[SD 11 — Arbre syntaxique abstrait (AST)]]).
```
Résultat : 1, 4, 7, 6, 3, 13, 14, 10, 8
```

**Par niveau (Level-order / BFS) — file FIFO**
Lecture niveau par niveau, plus court chemin non pondéré.
```
Résultat : 8, 3, 10, 1, 6, 14, 4, 7, 13
```

## Implémentation Python

```python
from collections import deque

class Node:
    def __init__(self, val): self.val = val; self.left = self.right = None

# Récursif — O(n) temps, O(h) espace
def inorder(n):   return inorder(n.left)   + [n.val] + inorder(n.right)   if n else []
def preorder(n):  return [n.val] + preorder(n.left)  + preorder(n.right)  if n else []
def postorder(n): return postorder(n.left) + postorder(n.right) + [n.val] if n else []

# Itératif in-order — évite le stack overflow sur arbres dégénérés
def inorder_iter(root):
    stack, result, cur = [], [], root
    while cur or stack:
        while cur:
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()
        result.append(cur.val)
        cur = cur.right
    return result

# BFS — niveau par niveau
def level_order(root):
    if not root: return []
    queue, result = deque([root]), []
    while queue:
        node = queue.popleft()
        result.append(node.val)
        if node.left:  queue.append(node.left)
        if node.right: queue.append(node.right)
    return result
```

## Complexités

| Parcours | Temps | Espace pile |
|---|---|---|
| Infixe / Préfixe / Suffixe | O(n) | O(h) — hauteur |
| BFS | O(n) | O(w) — largeur maximale |
| Arbre équilibré | — | O(log n) |
| Arbre dégénéré | — | O(n) |

> [!tip] Choisir le bon parcours
> BST → valeurs triées : **in-order**.
> Sérialiser / cloner la structure : **pre-order**.
> Supprimer / libérer / évaluer AST : **post-order**.
> Plus court chemin, reconstruction niveau par niveau : **BFS**.

> [!warning] Récursion sur arbre dégénéré
> Un arbre dégénéré (hauteur O(n)) peut provoquer un stack overflow avec la version récursive.
> Préférer la version itérative ou utiliser un arbre auto-équilibré dès la construction.
