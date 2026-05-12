#math #algebre-lineaire #matrices

## Matrice

Tableau rectangulaire de m lignes et n colonnes — notée M ∈ ℝᵐˣⁿ.

```
     ⎡ a₁₁  a₁₂  a₁₃ ⎤
A =  ⎢ a₂₁  a₂₂  a₂₃ ⎥   ∈ ℝ²ˣ³   (2 lignes, 3 colonnes)
     ⎣ a₃₁  a₃₂  a₃₃ ⎦
```

Notation : aᵢⱼ = élément ligne i, colonne j.

## Types de matrices

| Type | Définition |
|------|-----------|
| **Carrée** | m = n |
| **Identité Iₙ** | diagonale = 1, reste = 0 |
| **Nulle** | tous les éléments = 0 |
| **Diagonale** | aᵢⱼ = 0 si i ≠ j |
| **Triangulaire sup.** | aᵢⱼ = 0 si i > j |
| **Triangulaire inf.** | aᵢⱼ = 0 si i < j |
| **Symétrique** | A = Aᵀ, c'est-à-dire aᵢⱼ = aⱼᵢ |
| **Orthogonale** | AᵀA = I — colonnes orthonormées |

## Addition et soustraction

Opèrent **élément par élément** — matrices de même taille uniquement.

```
⎡1 2⎤   ⎡5 6⎤   ⎡ 6  8⎤
⎢3 4⎥ + ⎢7 8⎥ = ⎢10 12⎥
```

## Multiplication scalaire

```
    ⎡1 2⎤   ⎡3  6⎤
3 × ⎢3 4⎥ = ⎢9 12⎥
```

## Transposée

Échange lignes et colonnes : (Aᵀ)ᵢⱼ = Aⱼᵢ.

```
     ⎡1 2 3⎤          ⎡1 4⎤
A =  ⎢4 5 6⎥   Aᵀ =   ⎢2 5⎥
                       ⎣3 6⎦
```

Si A ∈ ℝᵐˣⁿ alors Aᵀ ∈ ℝⁿˣᵐ.

**Propriétés :**
- (Aᵀ)ᵀ = A
- (A + B)ᵀ = Aᵀ + Bᵀ
- (AB)ᵀ = BᵀAᵀ  ← l'ordre s'inverse

La matrice de Gram AᵀA est toujours symétrique et semi-définie positive.

## Rang

Le **rang** d'une matrice est le nombre de lignes (ou colonnes) linéairement indépendantes.

```
rang(A) ≤ min(m, n)
```

- **Plein rang** (rang = min(m,n)) → le système Ax = b a une solution unique (si carré).
- **Rang déficient** → lignes/colonnes redondantes, déterminant nul.

## Trace

Somme des éléments diagonaux (matrices carrées uniquement).

```
tr(A) = a₁₁ + a₂₂ + ... + aₙₙ
```

**Propriétés :**
- tr(AB) = tr(BA)
- tr(A) = Σλᵢ (somme des valeurs propres)

> [!tip] Matrice = transformation linéaire
> Une matrice m×n représente une fonction linéaire ℝⁿ → ℝᵐ.
> Multiplier Av transforme le vecteur v : rotations, mises à l'échelle, projections, cisaillements.
