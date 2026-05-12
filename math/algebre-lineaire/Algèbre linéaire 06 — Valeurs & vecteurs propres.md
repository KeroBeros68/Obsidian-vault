#math #algebre-lineaire #valeurs-propres #vecteurs-propres #eigenvalues

## Définition

λ est **valeur propre** et v est **vecteur propre** associé si :

```
Av = λv     (v ≠ 0)
```

La matrice A **ne change pas la direction** de v — elle l'étire ou le compresse d'un facteur λ.

```
A × v = λ × v
        ↑
   même direction, magnitude multipliée par λ
```

## Équation caractéristique

```
Av = λv
(A - λI)v = 0    ← système homogène
```

Ce système a une solution non nulle ssi det(A - λI) = 0.

**Polynôme caractéristique :**
```
p(λ) = det(A - λI) = 0
```

Pour A 2×2 :
```
     ⎡a b⎤
A =  ⎢c d⎥

det(A - λI) = (a-λ)(d-λ) - bc = λ² - tr(A)λ + det(A) = 0
```

**Exemple :**
```
A = [[3,1],[1,3]]

det(A - λI) = (3-λ)² - 1 = λ² - 6λ + 8 = (λ-2)(λ-4) = 0

λ₁ = 2,  λ₂ = 4
```

## Trouver les vecteurs propres

Pour chaque λᵢ, résoudre (A - λᵢI)v = 0 :

```
λ₁ = 2 :  (A - 2I)v = 0  →  ⎡1 1⎤ v = 0  →  v₁ = [1,-1] / √2
                              ⎢1 1⎥

λ₂ = 4 :  (A - 4I)v = 0  →  ⎡-1 1⎤ v = 0  →  v₂ = [1,1] / √2
                              ⎢ 1 -1⎥
```

Les vecteurs propres sont définis à un scalaire près (la direction compte, pas la longueur).

## Propriétés

| Propriété | Formule |
|-----------|---------|
| Somme des valeurs propres | Σλᵢ = tr(A) |
| Produit des valeurs propres | Πλᵢ = det(A) |
| Valeurs propres de Aᵀ | Identiques à celles de A |
| Valeurs propres de A⁻¹ | 1/λᵢ |
| Matrice symétrique | Valeurs propres réelles, vecteurs propres orthogonaux |
| Matrice définie positive | Toutes les valeurs propres > 0 |

## Diagonalisation

Si A a n vecteurs propres linéairement indépendants :

```
A = P D P⁻¹

P : matrice dont les colonnes sont les vecteurs propres
D : matrice diagonale des valeurs propres correspondantes

⎡λ₁ 0  0 ⎤
⎢0  λ₂ 0 ⎥
⎣0  0  λ₃⎦
```

**Puissance de matrice via diagonalisation :**
```
Aⁿ = P Dⁿ P⁻¹   — beaucoup plus rapide que n multiplications
```

**Matrice symétrique** (décomposition spectrale) :
```
A = Q Λ Qᵀ   (Q orthogonale — vecteurs propres orthonormés)
```

## Interprétations pratiques

```
Valeur propre λ :
  λ > 1  →  étirement dans la direction du vecteur propre
  0 < λ < 1  →  compression
  λ < 0  →  retournement
  λ = 0  →  effondrement (matrice singulière)

Applications :
  ACP (PCA)  →  vecteurs propres de la matrice de covariance = axes principaux
  PageRank   →  vecteur propre dominant du graphe de liens
  Stabilité  →  |λ| < 1 pour tous les i → système dynamique stable
```

> [!info] Valeurs propres complexes
> Pour une matrice réelle non symétrique, les valeurs propres peuvent être complexes (conjuguées par paires). Les matrices symétriques réelles ont toujours des valeurs propres réelles.
