#algo #arbres #bst #abr #insertion #suppression #recherche

## Structure de nœud

```python
class Node:
    def __init__(self, val: int) -> None:
        self.val   = val
        self.left  = self.right = None
```

## Recherche

```
si nœud == null      → non trouvé
si x == nœud.val     → trouvé
si x < nœud.val      → chercher à gauche
si x > nœud.val      → chercher à droite
```

```python
def search(root: Node | None, val: int) -> bool:
    while root:
        if val == root.val:   return True
        root = root.left if val < root.val else root.right
    return False
```

## Insertion

```
si nœud == null → créer feuille ← point d'insertion
si x < nœud.val → descendre à gauche
si x > nœud.val → descendre à droite
si x == nœud.val → doublon, ignorer
```

```python
def insert(root: Node | None, val: int) -> Node:
    if not root: return Node(val)
    if val < root.val:   root.left  = insert(root.left,  val)
    elif val > root.val: root.right = insert(root.right, val)
    return root
```

## Suppression — 3 cas

```
Cas 1 — feuille (0 enfant)
  → Supprimer directement, retourner null

Cas 2 — 1 enfant
  → Remplacer le nœud par son unique enfant

Cas 3 — 2 enfants  ← le plus délicat
  → Trouver le successeur infixe = min du sous-arbre droit
  → Copier sa valeur dans le nœud courant
  → Supprimer le successeur (qui a au plus 1 enfant droit)
```

```
Exemple — supprimer 3 (deux enfants : 1 et 6) :

        8                   8
       / \                 / \
      3   10    →         4   10
     / \    \            / \    \
    1   6    14         1   6    14
       / \   /              / \   /
      4   7 13             5   7 13

Successeur infixe de 3 = min(sous-arbre droit) = 4
→ nœud 3 prend la valeur 4, puis l'ancien nœud 4 est supprimé (cas 1 ou 2)
```

```python
def _find_min(node: Node) -> Node:
    while node.left: node = node.left
    return node

def delete(root: Node | None, val: int) -> Node | None:
    if not root: return None
    if val < root.val:
        root.left  = delete(root.left,  val)
    elif val > root.val:
        root.right = delete(root.right, val)
    else:
        if not root.left:  return root.right   # Cas 1 ou 2
        if not root.right: return root.left    # Cas 2
        # Cas 3 : successeur infixe
        successor  = _find_min(root.right)
        root.val   = successor.val
        root.right = delete(root.right, successor.val)
    return root
```

## Validation d'un BST — approche par bornes

```python
# ❌ Comparer uniquement parent/enfant direct ne suffit pas
# ✅ Propager les bornes autorisées à chaque descente
def is_valid_bst(node, lo=float('-inf'), hi=float('inf')) -> bool:
    if not node: return True
    if not (lo < node.val < hi): return False
    return (is_valid_bst(node.left,  lo,       node.val) and
            is_valid_bst(node.right, node.val, hi))
```

## Complexités

| Opération | BST équilibré | BST dégénéré |
|---|---|---|
| search | O(log n) | O(n) |
| insert | O(log n) | O(n) |
| delete | O(log n) | O(n) |
| min / max | O(log n) | O(n) |
| inorder | O(n) | O(n) |

> [!info] Lien avec SD 06
> [[SD 06 — Arbre binaire de recherche (ABR - BST)]] couvre la propriété et les interfaces.
> Cette fiche détaille la suppression cas 3, l'itération pour la recherche et la validation par bornes.

> [!warning] BST dégénéré — cas typique
> Insertions en ordre trié 1, 2, 3 … → hauteur O(n) → liste chaînée.
> En production : utiliser un arbre auto-équilibré ([[Arbres 03 — AVL — Rotations et équilibre]] ou [[Arbres 04 — Arbre rouge-noir]]).
