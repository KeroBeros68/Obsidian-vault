#math #algebre-lineaire #svd #lu #qr #décompositions

Les décompositions facteurisent une matrice en produit de matrices aux propriétés utiles.

## SVD — Décomposition en valeurs singulières

La plus générale. Fonctionne pour **toute matrice** m×n.

```
A = U Σ Vᵀ

U  : matrice orthogonale m×m  (vecteurs singuliers gauches)
Σ  : matrice diagonale m×n   (valeurs singulières σᵢ ≥ 0, décroissantes)
Vᵀ : matrice orthogonale n×n  (vecteurs singuliers droits)
```

```
Interprétation géométrique :
A = (rotation U) × (mise à l'échelle Σ) × (rotation Vᵀ)
```

**Valeurs singulières :** σᵢ = √(valeurs propres de AᵀA)

**Applications :**

| Application | Principe |
|-------------|---------|
| Rang | Nombre de valeurs singulières non nulles |
| Pseudo-inverse | A⁺ = V Σ⁺ Uᵀ |
| Approximation de rang k | Garder les k plus grandes valeurs singulières |
| Moindres carrés | Solution via la décomposition |

**Approximation de rang k :**
```
Aₖ = Σᵢ₌₁ᵏ σᵢ uᵢ vᵢᵀ   (somme des k termes de rang 1 dominants)
```
‖A - Aₖ‖ est minimale parmi toutes les matrices de rang k (théorème d'Eckart-Young).

## LU — Décomposition triangulaire

Pour matrices carrées.

```
PA = LU

P : matrice de permutation (échanges de lignes pour la stabilité numérique)
L : triangulaire inférieure avec diagonale = 1
U : triangulaire supérieure
```

```
Résoudre Ax = b via LU :
  1. PAx = Pb  →  LUx = Pb
  2. Résoudre Ly = Pb  (substitution avant — O(n²))
  3. Résoudre Ux = y   (substitution arrière — O(n²))
```

La factorisation elle-même coûte O(n³), mais permet de résoudre rapidement pour de nouveaux membres b.

## QR — Décomposition orthogonale-triangulaire

```
A = QR

Q : matrice orthogonale m×m (Qᵀ = Q⁻¹)
R : matrice triangulaire supérieure m×n
```

**Construction par algorithme de Gram-Schmidt :**
Orthogonalise les colonnes de A une par une pour former les colonnes de Q.

**Applications :**
- Résolution stable des moindres carrés : min ‖Ax - b‖² → résoudre Rx = Qᵀb
- Algorithme QR itératif pour calculer toutes les valeurs propres
- Orthogonalisation de base

## Cholesky — matrices définies positives

Pour matrices **symétriques définies positives** (SDP) uniquement.

```
A = LLᵀ    (L triangulaire inférieure à diagonale positive)
```

Existence : A est SDP ⟺ toutes les valeurs propres sont > 0 ⟺ la décomposition de Cholesky existe.

Deux fois moins d'opérations que LU. Utilisée dans les simulations Monte Carlo, les modèles gaussiens, les filtres de Kalman.

## Comparaison des décompositions

| Décomposition | Matrices | Usage principal |
|---|---|---|
| **LU** | Carrées | Résoudre Ax = b, calculer det(A) |
| **QR** | Quelconques | Moindres carrés stables, valeurs propres |
| **SVD** | Quelconques | Rang, pseudo-inverse, approximation de rang k |
| **Cholesky** | SDP | Systèmes SDP, simulation |
| **Spectrale** | Symétriques | Diagonalisation, ACP |

> [!tip] SVD = décomposition universelle
> SVD fonctionne dans tous les cas et révèle la structure géométrique complète d'une matrice. LU et Cholesky sont plus efficaces quand les conditions d'application sont réunies.
