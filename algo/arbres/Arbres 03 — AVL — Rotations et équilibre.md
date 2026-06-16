#algo #arbres #avl #équilibré #rotations

## Définition

Un AVL est un [[Arbres 02 — BST — Opérations complètes|BST]] auto-équilibré :

```
facteur d'équilibre (bf) = hauteur(sous-arbre gauche) − hauteur(sous-arbre droit)

Invariant : bf ∈ {-1, 0, +1} pour chaque nœud
→ garantit hauteur h ≤ 1.44 · log₂(n+1) → toutes opérations O(log n)
```

## Les quatre rotations

**Rotation droite — cas LL (déséquilibre gauche-gauche)**
```
    z (bf=+2)           y
   /            →      / \
  y (bf=+1)           x   z
 /
x
```

**Rotation gauche — cas RR (déséquilibre droite-droite)**
```
  z (bf=-2)                y
    \           →          / \
     y (bf=-1)            z   x
       \
        x
```

**Rotation gauche-droite — cas LR**
```
Étape 1 : rotation gauche sur y → cas LL
Étape 2 : rotation droite sur z

    z (bf=+2)        z (bf=+2)          x
   /          →     /           →      / \
  y (bf=-1)        x (bf=+1)          y   z
    \              /
     x            y
```

**Rotation droite-gauche — cas RL**
```
Étape 1 : rotation droite sur y → cas RR
Étape 2 : rotation gauche sur z

  z (bf=-2)         z (bf=-2)           x
    \        →        \        →       / \
     y (bf=+1)         x (bf=-1)      z   y
    /                    \
   x                      y
```

## Règle de déclenchement

| bf du nœud déséquilibré | Position du nouveau nœud | Rotation |
|---|---|---|
| bf > +1 | val < gauche.val | Simple droite (LL) |
| bf > +1 | val > gauche.val | Gauche-droite (LR) |
| bf < −1 | val > droite.val | Simple gauche (RR) |
| bf < −1 | val < droite.val | Droite-gauche (RL) |

## Implémentation Python

```python
class AVLNode:
    def __init__(self, val):
        self.val = val; self.left = self.right = None; self.height = 1

def _h(n):   return n.height if n else 0
def _bf(n):  return _h(n.left) - _h(n.right) if n else 0
def _upd(n): n.height = 1 + max(_h(n.left), _h(n.right))

def _rot_right(z):
    y = z.left; T3 = y.right
    y.right = z; z.left = T3
    _upd(z); _upd(y)
    return y  # nouvelle racine du sous-arbre

def _rot_left(z):
    y = z.right; T2 = y.left
    y.left = z; z.right = T2
    _upd(z); _upd(y)
    return y

def avl_insert(root, val):
    if not root: return AVLNode(val)
    if val < root.val:   root.left  = avl_insert(root.left,  val)
    elif val > root.val: root.right = avl_insert(root.right, val)
    else:                return root   # doublon ignoré
    _upd(root)
    bf = _bf(root)
    if bf >  1 and val < root.left.val:   return _rot_right(root)          # LL
    if bf < -1 and val > root.right.val:  return _rot_left(root)           # RR
    if bf >  1 and val > root.left.val:                                     # LR
        root.left = _rot_left(root.left);  return _rot_right(root)
    if bf < -1 and val < root.right.val:                                    # RL
        root.right = _rot_right(root.right); return _rot_left(root)
    return root
```

## Complexités

| Opération | AVL |
|---|---|
| search | O(log n) |
| insert | O(log n) — ≤ 2 rotations |
| delete | O(log n) — O(log n) rotations en remontant |
| Hauteur garantie | ≤ 1.44 · log₂(n+1) |

> [!tip] AVL vs Rouge-Noir
> AVL est plus strict → hauteur plus faible → lectures plus rapides.
> Rouge-Noir tolère plus de déséquilibre → moins de rotations à l'écriture.
> AVL si lectures >> écritures. Rouge-Noir si écritures fréquentes.
> En pratique, les stdlib (Java, C++) choisissent [[Arbres 04 — Arbre rouge-noir]].

> [!warning] Suppression AVL
> La suppression peut déclencher des rééquilibrages en cascade jusqu'à la racine.
> Implémenter le rebalancement récursivement sur le chemin de retour, comme pour l'insertion.
