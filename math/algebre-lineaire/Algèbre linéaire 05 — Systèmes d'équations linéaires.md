#math #algebre-lineaire #systemes-lineaires #gauss

## Forme matricielle

Un système de m équations à n inconnues s'écrit **Ax = b** :

```
a₁₁x₁ + a₁₂x₂ + a₁₃x₃ = b₁
a₂₁x₁ + a₂₂x₂ + a₂₃x₃ = b₂     →    Ax = b
a₃₁x₁ + a₃₂x₂ + a₃₃x₃ = b₃
```

```
Exemple : 2x + y - z = 8
         -3x - y + 2z = -11    →  x = 2, y = 3, z = -1
         -2x + y + 2z = -3
```

## Types de solutions

```
det(A) ≠ 0  →  solution unique    x = A⁻¹b

det(A) = 0  →  soit aucune solution (système incompatible)
              soit infinité de solutions (équations redondantes)
```

```
Géométrie en ℝ² (2 équations, 2 inconnues = 2 droites) :

Unique   Aucune    Infini
  ╲ ╱      ║ ║      ║
   ╳        ║ ║      ║  (même droite)
  ╱ ╲      ║ ║
```

## Élimination de Gauss

Transforme Ax = b en une forme triangulaire par opérations élémentaires sur les lignes :

1. Échange de deux lignes
2. Multiplication d'une ligne par un scalaire ≠ 0
3. Ajout d'un multiple d'une ligne à une autre

```
⎡ 2  1 -1 |  8 ⎤      ⎡2  1  -1 |  8 ⎤
⎢-3 -1  2 |-11 ⎥  →   ⎢0  0.5 0.5| 1 ⎥   (pivot sur colonne 1)
⎢-2  1  2 | -3 ⎥      ⎢0  2   1 |  5 ⎥

→ forme triangulaire → substitution arrière
```

La décomposition LU (fiche 07) est une variante systématique de Gauss.

## Système sur-déterminé (m > n) — moindres carrés

Plus d'équations que d'inconnues : en général pas de solution exacte.
On cherche x minimisant **‖Ax - b‖²** :

```
x* = (AᵀA)⁻¹Aᵀb   (solution moindres carrés)
   = A⁺b           (via pseudo-inverse)
```

Interprétation : projection orthogonale de b sur l'espace colonne de A.

## Système sous-déterminé (m < n) — infinité de solutions

Moins d'équations que d'inconnues. Parmi toutes les solutions, la pseudo-inverse donne celle de **norme minimale** :

```
x* = A⁺b = Aᵀ(AAᵀ)⁻¹b
```

## Interprétation géométrique

```
Ax = b  →  b est-il dans l'espace colonne de A ?

     espace colonne de A
     ╔═════════════╗
     ║             ║
     ║    b*       ║ ← projection de b (si b hors de l'espace)
     ║             ║
     ╚═════════════╝
               ↑
               b (hors de l'espace → moindres carrés)
```

## Rang et solutions

| rang(A) | rang([A|b]) | Solutions |
|---------|-------------|-----------|
| n (plein rang colonnes) | n | Solution unique (si m=n) |
| < n | = rang(A) | Infinité de solutions |
| < rang([A|b]) | > rang(A) | Aucune solution |

> [!tip] Théorème du rang (rank-nullity)
> Pour A ∈ ℝᵐˣⁿ : rang(A) + dim(noyau(A)) = n
> Le noyau (kernel) est l'ensemble des x tels que Ax = 0.
