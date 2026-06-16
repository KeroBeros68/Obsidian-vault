#algo #arbres #rouge-noir #red-black #équilibré

## Les 5 invariants

```
1. Chaque nœud est ROUGE ou NOIR.
2. La racine est toujours NOIRE.
3. Les feuilles (sentinelles NIL) sont NOIRES.
4. Tout nœud ROUGE a deux enfants NOIRS (pas de deux rouges consécutifs).
5. Tout chemin racine→feuille traverse le même nombre de nœuds NOIRS (black-height).
```

Ces invariants garantissent : hauteur h ≤ 2 · log₂(n+1) → O(log n) garanti.

## Structure

```
         B:8
        /     \
     R:3       B:10
    /   \          \
  B:1   B:6        R:14
        / \        /
      R:4  R:7   R:13

(B = noir, R = rouge — sentinelles NIL non représentées)
```

## Insertion — règle de départ + 4 cas

**Tout nouveau nœud inséré est ROUGE.**

```
Cas 1 — Le nœud inséré est la racine
  → Recolorer en NOIR. Fin.

Cas 2 — Le parent est NOIR
  → Aucune violation. Fin.

Cas 3 — Parent ROUGE, oncle ROUGE (recoloration)
  → Parent + oncle → NOIR
  → Grand-parent → ROUGE
  → Répéter depuis le grand-parent (remonter vers la racine)

Cas 4a — Parent ROUGE, oncle NOIR, triangle (LR ou RL)
  → Rotation sur le parent pour former une ligne → appliquer cas 4b

Cas 4b — Parent ROUGE, oncle NOIR, ligne (LL ou RR)
  → Rotation sur le grand-parent
  → Échanger les couleurs parent ↔ grand-parent
```

## Diagramme des cas clés

```
Cas 3 — Recoloration :
      G(N)                G(R) ← remonter ici
      /  \                /  \
   P(R)  U(R)    →    P(N)  U(N)
   /                  /
 N(R)               N(R)

Cas 4b — Ligne LL → rotation droite :
      G(N)                P(N)
      /  \                /  \
   P(R)  U(N)    →    N(R)  G(R)
   /                           \
 N(R)                          U(N)
```

## Utilisations dans les langages standards

| Langage | Collection | Structure interne |
|---|---|---|
| Java | `TreeMap<K,V>`, `TreeSet<E>` | Rouge-Noir |
| C++ STL | `std::map`, `std::set`, `std::multimap` | Rouge-Noir |
| Linux kernel | `rbtree.h` — scheduler CFS | Rouge-Noir |
| Rust std | `BTreeMap`, `BTreeSet` | B-Tree (pas RB) |
| Python | `sortedcontainers.SortedList` | Skip-list (pas RB) |

```java
// Java — TreeMap = rouge-noir interne
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(5, "cinq"); tm.put(3, "trois"); tm.put(8, "huit");
tm.firstKey();       // 3  — O(log n)
tm.lastKey();        // 8  — O(log n)
tm.floorKey(4);      // 3  — plus grand ≤ 4
tm.ceilingKey(6);    // 8  — plus petit ≥ 6
tm.subMap(3, 8);     // [3, 8[ — O(log n + k)
```

```cpp
// C++ — std::map = rouge-noir interne
#include <map>
std::map<int, std::string> m;
m[5] = "cinq"; m[3] = "trois";
m.begin()->first;          // min
m.rbegin()->first;         // max
m.lower_bound(4)->first;   // premier ≥ 4
```

## Complexités

| Opération | Rouge-Noir |
|---|---|
| search | O(log n) |
| insert | O(log n) — ≤ 2 rotations |
| delete | O(log n) — ≤ 3 rotations |
| Hauteur max | 2 · log₂(n+1) |

> [!tip] Pourquoi les stdlib préfèrent le rouge-noir à l'AVL
> Moins de rotations à l'insertion/suppression que l'[[Arbres 03 — AVL — Rotations et équilibre|AVL]].
> L'AVL est plus strict (lectures plus rapides) mais plus coûteux en écriture.
> Pour les collections générales (mix lecture/écriture), le rouge-noir est le meilleur compromis.

> [!warning] Ne pas réimplémenter soi-même
> La suppression rouge-noir comporte 6 sous-cas imbriqués.
> En production, utiliser `TreeMap` / `std::map`.
> Réimplémenter uniquement en exercice algorithmique ou contexte sans stdlib.
