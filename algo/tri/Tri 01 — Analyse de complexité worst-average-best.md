#algo #tri #complexité #bases

## Analyse de complexité — worst / average / best

La complexité d'un algorithme de tri se mesure en **nombre de comparaisons** (ou d'opérations élémentaires) en fonction de la taille n du tableau.

## Les trois cas à distinguer

| Cas | Signification | Exemple concret |
|-----|---------------|-----------------|
| **Best case** | Entrée la plus favorable possible | Tableau déjà trié |
| **Average case** | Entrée aléatoire (espérance) | Permutation uniforme |
| **Worst case** | Entrée la plus défavorable possible | Tableau trié à l'envers |

## Tableau comparatif global

| Algorithme | Best | Average | Worst | Espace | Stable |
|------------|------|---------|-------|--------|--------|
| **Tri fusion** | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ |
| **Tri rapide** | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ |
| **Tri par tas** | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ |
| **Tri comptage** | O(n + k) | O(n + k) | O(n + k) | O(k) | ✅ |
| **Tri radix** | O(d·(n+k)) | O(d·(n+k)) | O(d·(n+k)) | O(n+k) | ✅ |
| Tri insertion | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Tri sélection | O(n²) | O(n²) | O(n²) | O(1) | ❌ |
| Tri bulles | O(n) | O(n²) | O(n²) | O(1) | ✅ |

*k = étendue des valeurs, d = nombre de chiffres (radix)*

## La borne inférieure Ω(n log n)

Tout algorithme de tri **par comparaisons** ne peut pas faire mieux que O(n log n) dans le cas moyen.

```
Preuve par arbre de décision :
- n éléments → n! permutations possibles
- Chaque comparaison binaire coupe les cas en 2
- Hauteur minimale de l'arbre = ⌈log₂(n!)⌉
- Par l'approximation de Stirling : log₂(n!) ≈ n log₂ n
→ Au moins n log n comparaisons nécessaires. ✅
```

Les tris par comptage et radix **échappent à cette borne** car ils ne comparent pas les éléments entre eux.

## Stabilité — définition

Un tri est **stable** si deux éléments de valeur égale conservent leur ordre relatif d'origine.

```python
data = [('Bob', 2), ('Alice', 1), ('Charlie', 2)]

# Après tri stable par valeur :
# [('Alice', 1), ('Bob', 2), ('Charlie', 2)]
#                 ↑                ↑
#           Bob avant Charlie — ordre original respecté ✅

# Après tri instable :
# [('Alice', 1), ('Charlie', 2), ('Bob', 2)]  ← peut arriver ❌
```

## Tri en place vs tri externe

```
En place  : O(1) espace auxiliaire. Modifie le tableau original.
            → Tri rapide, tri par tas, tri insertion.

Externe   : O(n) ou plus d'espace auxiliaire. Crée des tableaux temporaires.
            → Tri fusion (O(n)), tri comptage (O(k)), tri radix (O(n+k)).

Externe stable + O(n log n) worst : tri fusion — le choix par défaut en pratique.
```

## Quand utiliser quel tri

| Situation | Tri recommandé | Raison |
|-----------|---------------|--------|
| Usage général | Tri rapide (introsort) | Rapide en pratique, O(log n) espace |
| Stabilité requise | Tri fusion | Stable + O(n log n) garanti |
| Mémoire contrainte | Tri par tas | O(1) espace, O(n log n) garanti |
| Petits entiers bornés | Tri comptage | O(n + k) si k raisonnable |
| Grands entiers / chaînes | Tri radix | O(d·n) indépendant des comparaisons |
| Tableau presque trié | Tri insertion | O(n) sur données quasi-triées |

> [!tip] Règle pratique
> En compétition ou en entretien : tri fusion si stabilité requise, tri rapide sinon. En production : utiliser le tri natif du langage (`sorted()` Python = Timsort, fusion + insertion hybride).

> [!info] Timsort
> Python, Java et Android utilisent **Timsort** — un hybride tri fusion + tri insertion qui détecte les sous-séquences déjà triées. Complexité : O(n) best, O(n log n) worst, stable.
