#math #algebre-lineaire #produit-matriciel

## Règle de compatibilité

Pour calculer AB, le nombre de colonnes de A doit égaler le nombre de lignes de B.

```
A        ×    B      =    C
(m × k)      (k × n)     (m × n)
```

La dimension intérieure (k) doit coïncider. Le résultat a les dimensions extérieures.

```
(2×3) × (3×4) = (2×4)   ✅
(2×3) × (2×4) = ❌       (3 ≠ 2)
```

## Calcul — produit ligne par colonne

Cᵢⱼ = ligne i de A · colonne j de B (produit scalaire) :

```
     ⎡1 2⎤         ⎡5 6⎤
A =  ⎢3 4⎥   B =   ⎢7 8⎥

C₁₁ = [1,2]·[5,7] = 1×5 + 2×7 = 19
C₁₂ = [1,2]·[6,8] = 1×6 + 2×8 = 22
C₂₁ = [3,4]·[5,7] = 3×5 + 4×7 = 43
C₂₂ = [3,4]·[6,8] = 3×6 + 4×8 = 50

     ⎡19 22⎤
C =  ⎢43 50⎥
```

## Propriétés

| Propriété | Formule | Remarque |
|-----------|---------|---------|
| Associativité | (AB)C = A(BC) | ✅ |
| Distributivité | A(B+C) = AB + AC | ✅ |
| **Non-commutativité** | AB ≠ BA en général | ⚠️ l'ordre compte |
| Élément neutre | AI = IA = A | I = matrice identité |
| Transposée | (AB)ᵀ = BᵀAᵀ | ordre inversé |
| Nulle | A × 0 = 0 | |

## Multiplication par un vecteur

Cas particulier : B est un vecteur colonne (n×1).

```
     ⎡1 2 3⎤   ⎡x⎤   ⎡1x + 2y + 3z⎤
Av = ⎢4 5 6⎥ × ⎢y⎥ = ⎢4x + 5y + 6z⎥
               ⎣z⎦
```

Chaque ligne de A définit une équation linéaire. Av = b est un **système linéaire**.

## Interprétation géométrique

Multiplier par une matrice applique une **transformation linéaire** :

```
Rotation 90° (ℝ²) :      Mise à l'échelle :    Projection sur x :
⎡0 -1⎤                   ⎡2 0⎤                 ⎡1 0⎤
⎢1  0⎥                   ⎢0 3⎥                 ⎢0 0⎥

[1,0] → [0,1]             [1,0] → [2,0]         [1,1] → [1,0]
[0,1] → [-1,0]            [0,1] → [0,3]
```

## Produit de Hadamard (élément par élément)

Distinct du produit matriciel — noté ⊙ :

```
     ⎡1 2⎤ ⊙ ⎡5 6⎤ = ⎡1×5 2×6⎤ = ⎡5  12⎤
     ⎢3 4⎥   ⎢7 8⎥   ⎢3×7 4×8⎥   ⎢21 32⎥
```

Utilisé en deep learning (masques, attention, gradient par élément).

## Puissance matricielle

```
A² = A × A
A³ = A × A × A
A⁰ = I
A⁻¹ = matrice inverse (si elle existe)
```

**Via diagonalisation** (si A = PDP⁻¹) :
```
Aⁿ = P Dⁿ P⁻¹
```
Beaucoup plus rapide pour les grandes puissances.

> [!tip] Coût du produit matriciel
> Multiplier (m×k) par (k×n) coûte O(m×k×n) opérations. Pour une chaîne ABC, l'ordre de parenthésage change le nombre d'opérations sans changer le résultat.
