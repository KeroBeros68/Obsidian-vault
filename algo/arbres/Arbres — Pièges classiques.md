#algo #arbres #pièges #erreurs #debugging

## 🪤 Piège 1 — BST dégénéré sur données triées

```python
root = None
for i in range(1, 1001):       # ❌ hauteur = 1000 → O(n) partout
    root = insert(root, i)

# ✅ Option 1 : mélanger avant d'insérer (données aléatoires → h ≈ log n)
import random
vals = list(range(1, 1001)); random.shuffle(vals)

# ✅ Option 2 : utiliser un arbre auto-équilibré
from sortedcontainers import SortedList
sl = SortedList(range(1, 1001))   # O(log n) garanti
```

> [!warning] Cas typique en entretien
> Construire un BST depuis un tableau trié = liste chaînée déguisée.
> Mentionner ce cas chaque fois qu'on discute des complexités d'un BST.

---

## 🪤 Piège 2 — Validation BST par comparaison parent direct

```python
# ❌ FAUX : ne vérifie que l'enfant immédiat, pas toute la sous-hiérarchie
def is_bst_wrong(node):
    if not node: return True
    if node.left  and node.left.val  >= node.val: return False
    if node.right and node.right.val <= node.val: return False
    return is_bst_wrong(node.left) and is_bst_wrong(node.right)

# Contre-exemple : arbre 5 → gauche:3 → droite:6
# 6 > 3 ✅ (test local) mais 6 > 5 ❌ (doit être < 5 dans le sous-arbre gauche)

# ✅ CORRECT : propager les bornes autorisées
def is_valid_bst(node, lo=float('-inf'), hi=float('inf')) -> bool:
    if not node: return True
    if not (lo < node.val < hi): return False
    return (is_valid_bst(node.left,  lo,       node.val) and
            is_valid_bst(node.right, node.val, hi))
```

> [!tip] Règle mémo
> La propriété BST n'est pas locale (parent/enfant direct) — elle est globale sur toute la sous-hiérarchie.
> Toujours propager des bornes `(lo, hi)` dans la récursion.

---

## 🪤 Piège 3 — Confondre in-order et pre-order pour reconstruire un arbre

```python
# ❌ In-order seul ne suffit pas à reconstruire la structure
# → retourne les valeurs triées, pas la topologie

# ✅ Pour reconstruire un BST identique :
#    Sérialiser en pre-order → réinsérer dans le même ordre
#    (chaque valeur descend au bon endroit)

# ✅ Pour reconstruire un arbre général :
#    Utiliser pre-order + in-order ensemble (ou post-order + in-order)
```

> [!tip] Règle mémo
> In-order → valeurs triées (utile pour vérifier, pas pour reconstruire).
> Pre-order → structure (premier élément = racine, récursivement).

---

## 🪤 Piège 4 — heapq avec objets non comparables

```python
import heapq

class Task: pass
heap = []
heapq.heappush(heap, (5, Task()))
heapq.heappush(heap, (5, Task()))   # ❌ TypeError: '<' not supported
# Si deux priorités sont identiques, Python compare le 2e élément du tuple.
# Task() n'implémente pas __lt__ → crash.

# ✅ Tie-breaker avec un compteur unique
import itertools
counter = itertools.count()
heapq.heappush(heap, (5, next(counter), Task()))
heapq.heappush(heap, (5, next(counter), Task()))   # ✅
```

> [!tip] Règle systématique
> Dès qu'on insère des objets dans un heap Python, insérer un compteur entre la priorité et l'objet.

---

## 🪤 Piège 5 — Confusion hauteur / profondeur

```
Profondeur d'un nœud : distance à la racine. Racine = 0.
Hauteur d'un nœud    : longueur du chemin le plus long vers une feuille.
Hauteur de l'arbre   : hauteur de la racine.

Arbre parfait à 7 nœuds : hauteur de la racine = 2, profondeur des feuilles = 2.

⚠️ Convention hauteur de l'arbre vide : −1 ou 0 selon les sources — vérifier avant d'implémenter.
```

> [!warning] Impact sur le facteur d'équilibre AVL
> Mélanger hauteur et profondeur dans le calcul du facteur d'équilibre produit un déséquilibre silencieux.
> L'arbre semble valide mais les invariants AVL sont violés.

---

## 🪤 Piège 6 — heapreplace sur un heap vide

```python
import heapq

heap = []
result = heapq.heapreplace(heap, 5)   # ❌ IndexError

result = heapq.heappushpop(heap, 5)   # ✅ retourne 5 sans crash si heap vide
```

> [!tip] Règle mémo
> `heapreplace` : plus rapide, mais raise si vide. Utiliser quand on est sûr que le heap n'est pas vide.
> `heappushpop` : plus sûr, légèrement moins rapide. Utiliser en cas de doute.

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| BST dégénéré sur données triées | Utiliser AVL / Rouge-Noir / SortedList |
| Validation BST par parent direct | Propager les bornes `(lo, hi)` récursivement |
| Reconstruction depuis in-order seul | Utiliser pre-order + in-order ensemble |
| Objets non-comparables dans heapq | Ajouter un tie-breaker `itertools.count()` |
| Confusion hauteur / profondeur | Hauteur = vers les feuilles, profondeur = vers la racine |
| `heapreplace` sur heap vide | Préférer `heappushpop` si le heap peut être vide |
