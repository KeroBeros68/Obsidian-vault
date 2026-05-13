#algo #tri #cocktail-sort #classique

## Tri cocktail — Cocktail Shaker Sort

Variante bidirectionnelle du tri à bulles : alterne les passes **gauche→droite** (remonte les grands) et **droite→gauche** (descend les petits). Réduit le problème des "tortues".

## Propriétés

- **Stable** : aucun swap entre éléments égaux.
- **En place** : O(1) espace auxiliaire.
- Même complexité asymptotique que bubble sort : O(n²) average et worst.
- Meilleur que bubble sort en pratique grâce à la bidirectionnalité — réduit les passes nécessaires.

## Problème des tortues (motivation)

```
Bubble sort classique (gauche→droite) :
  - Les grands éléments remontent vite (une passe suffit).
  - Les petits éléments en fin de tableau descendent lentement (une position par passe).

Exemple : [2, 3, 4, 5, 1]
  Bubble sort : 4 passes pour amener 1 en position 0. 🐢
  Cocktail    : la passe droite→gauche amène 1 en position 0 dès la 1ère passe. ✅
```

## Implémentation

```python
def cocktail_sort(arr):
    n       = len(arr)
    left    = 0
    right   = n - 1
    swapped = True

    while swapped and left < right:
        swapped = False

        # Passe gauche → droite : remonte le max ✅
        for i in range(left, right):
            if arr[i] > arr[i + 1]:
                arr[i], arr[i + 1] = arr[i + 1], arr[i]
                swapped = True
        right -= 1                           # ✅ dernier élément en place

        if not swapped:
            break

        swapped = False
        # Passe droite → gauche : descend le min ✅
        for i in range(right, left, -1):
            if arr[i] < arr[i - 1]:
                arr[i], arr[i - 1] = arr[i - 1], arr[i]
                swapped = True
        left += 1                            # ✅ premier élément en place

    return arr
```

## Illustration — une itération complète

```
Tableau : [5, 1, 4, 2, 8, 0, 2]

Passe →  : [1,4,2,5,0,2,8]  ← 8 en place à droite
Passe ←  : [0,1,4,2,5,2,8]  ← 0 en place à gauche

Frontières rétrécissent : left=1, right=5
Passe →  : [0,1,2,4,2,5,8]
Passe ←  : [0,1,2,4,2,5,8]  ← aucun swap dans la zone
...
Final : [0,1,2,2,4,5,8] ✅
```

## Complexité

| Cas | Quand | Temps |
|-----|-------|-------|
| Best | Tableau trié | O(n) |
| Average | Permutation aléatoire | O(n²) |
| Worst | Tableau inversé | O(n²) |

## Comparaison Bubble vs Cocktail

| Critère | Bubble Sort | Cocktail Sort |
|---------|-------------|---------------|
| Passes pour trier | n−1 | ~n/2 (en pratique) |
| Tortues (petits en fin) | Lentes 🐢 | Corrigées ✅ |
| Lapins (grands en début) | Rapides ✅ | Rapides ✅ |
| Complexité asymptotique | O(n²) | O(n²) |
| Stable | ✅ | ✅ |
| Implémentation | Simple | Légèrement plus complexe |

> [!tip] Analogue à un barman
> Cocktail sort secoue le tableau dans les deux sens comme un shaker — d'où son nom alternatif "Cocktail Shaker Sort".

> [!info] Variantes proches
> **Odd-even sort** (tri pair-impair) : trie alternativement les paires (0,1),(2,3),... puis (1,2),(3,4),...  
> **Comb sort** : comme cocktail mais avec un gap décroissant (facteur ≈1.3) → O(n log n) en moyenne.
