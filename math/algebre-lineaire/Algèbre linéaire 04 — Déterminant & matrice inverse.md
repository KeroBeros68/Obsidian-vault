#math #algebre-lineaire #déterminant #inverse

## Déterminant

Scalaire associé à une **matrice carrée**, mesurant le facteur d'expansion de volume de la transformation.

**Matrice 2×2 :**
```
     ⎡a b⎤
A =  ⎢c d⎥     det(A) = ad - bc
```

```
det([1 2; 3 4]) = 1×4 - 2×3 = 4 - 6 = -2
```

**Matrice 3×3 — développement selon la première ligne :**
```
det(A) = a₁₁(a₂₂a₃₃ - a₂₃a₃₂)
       - a₁₂(a₂₁a₃₃ - a₂₃a₃₁)
       + a₁₃(a₂₁a₃₂ - a₂₂a₃₁)
```

Le signe alterne : +, −, +, −, ... (règle du damier).

## Interprétation géométrique

```
det = 0   →  transformation aplatie (perte de dimension)
             les colonnes sont linéairement dépendantes

|det| > 1  →  expansion de volume
|det| < 1  →  contraction de volume
det < 0   →  retournement d'orientation
```

```
det([1,0],[0,1]) = 1   (carré unité → carré unité)
det([2,0],[0,3]) = 6   (carré unité → rectangle 2×3)
det([1,1],[0,0]) = 0   (carré → segment aplati)
```

## Propriétés du déterminant

| Propriété | Formule |
|-----------|---------|
| Produit | det(AB) = det(A) × det(B) |
| Transposée | det(Aᵀ) = det(A) |
| Inverse | det(A⁻¹) = 1 / det(A) |
| Scalaire | det(αA) = αⁿ × det(A) pour A ∈ ℝⁿˣⁿ |
| Permutation de lignes | Change le signe |
| Ligne nulle ou dupliquée | det = 0 |

## Matrice inverse

A⁻¹ est la matrice telle que : **A × A⁻¹ = A⁻¹ × A = I**

**Existence :** A est inversible ⟺ det(A) ≠ 0 ⟺ rang(A) = n.

**Formule 2×2 :**
```
     ⎡a b⎤⁻¹        1     ⎡ d -b⎤
     ⎢c d⎥    =  ——————— × ⎢-c  a⎥
                  ad - bc
```

**Via la matrice des cofacteurs (formule de Cramer) :**
```
A⁻¹ = (1 / det(A)) × Cᵀ

C = matrice des cofacteurs
Cᵢⱼ = (-1)^(i+j) × det(A sans ligne i et colonne j)
```

> [!warning] Calculer A⁻¹ pour résoudre Ax = b est inefficace
> La résolution directe par élimination de Gauss (fiche 05) est plus rapide et plus stable numériquement que le calcul explicite de l'inverse.

## Matrice pseudo-inverse (Moore-Penrose)

Généralise l'inverse aux matrices non carrées ou singulières.

```
A⁺ = (AᵀA)⁻¹Aᵀ   (si A a des colonnes indépendantes)
```

Utilisée pour les moindres carrés : minimise ‖Ax - b‖².

```
Propriétés :
  AA⁺A  = A
  A⁺AA⁺ = A⁺
  (AA⁺)ᵀ = AA⁺
  (A⁺A)ᵀ = A⁺A
```

## Matrice orthogonale

Q est orthogonale si Qᵀ = Q⁻¹, c'est-à-dire QᵀQ = I.

- Colonnes orthonormées : qᵢ · qⱼ = 0 si i≠j, 1 si i=j
- Préserve les normes : ‖Qv‖ = ‖v‖
- det(Q) = ±1
- Exemples : matrices de rotation, réflexions

## Nombre de condition

```
κ(A) = ‖A‖ × ‖A⁻¹‖ = σ_max / σ_min
```

Mesure la sensibilité de la solution x à de petites perturbations dans A ou b :
- κ ≈ 1 → bien conditionné, inversion stable
- κ ≫ 1 → mal conditionné, amplification des erreurs

> [!tip] Matrice singulière vs mal conditionnée
> Une matrice singulière a det = 0 exactement — pas d'inverse.
> Une matrice mal conditionnée a det ≈ 0 — l'inverse existe mais est numériquement instable.
