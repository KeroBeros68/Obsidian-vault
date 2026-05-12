#math #algebre-lineaire #pièges #erreurs

## 🪤 Piège 1 — Non-commutativité du produit matriciel

```
AB ≠ BA   en général

Exemple :
A = [[1,2],[0,1]]   B = [[1,0],[1,1]]

AB = [[3,2],[1,1]]
BA = [[1,2],[1,3]]   ≠ AB
```

L'ordre compte dans toute formule impliquant un produit. En particulier :
```
(AB)ᵀ = BᵀAᵀ   ← ordre inversé, pas AᵀBᵀ
(ABC)⁻¹ = C⁻¹B⁻¹A⁻¹   ← ordre inversé
```

---

## 🪤 Piège 2 — Confondre déterminant nul et matrice nulle

```
det(A) = 0   ≠   A = 0

Exemple : A = [[1,2],[2,4]]
det(A) = 1×4 - 2×2 = 0   mais A ≠ 0
```

det(A) = 0 signifie que les lignes (ou colonnes) sont **linéairement dépendantes** — pas que A est nulle.

---

## 🪤 Piège 3 — Croire qu'une matrice carrée est toujours inversible

```
A inversible  ⟺  det(A) ≠ 0  ⟺  rang(A) = n  ⟺  aucune valeur propre nulle

Contre-exemple :
A = [[1,2],[2,4]]   →   det = 0   →   non inversible
```

Avant d'utiliser A⁻¹ dans un raisonnement, vérifier que det(A) ≠ 0.

---

## 🪤 Piège 4 — Confondre vecteur propre et valeur propre

```
Av = λv

λ (scalaire) : valeur propre — facteur d'étirement
v (vecteur)  : vecteur propre — direction invariante
```

Un vecteur propre n'est pas unique : αv est aussi vecteur propre pour tout α ≠ 0.
Deux valeurs propres distinctes donnent des vecteurs propres orthogonaux (si A symétrique).

---

## 🪤 Piège 5 — Incompatibilité de dimensions dans un produit

```
A (m×k) × B (k×n) = C (m×n)   ✅   dimension intérieure commune

A (2×3) × B (2×3) = ❌   (3 ≠ 2)
A (3×2) × B (2×3) = ✅   (2 = 2)   →  résultat (3×3)
A (2×3) × B (3×2) = ✅   (3 = 3)   →  résultat (2×2)
```

AᵀA et AAᵀ sont toujours bien définies — mais ce ne sont pas la même matrice.

---

## 🪤 Piège 6 — Moindres carrés vs solution exacte

```
Système sur-déterminé : m > n  →  pas de solution exacte en général
                               →  moindres carrés : min ‖Ax - b‖²

Confondre :
  x = A⁻¹b          ← valide uniquement si A est carrée et inversible
  x = (AᵀA)⁻¹Aᵀb   ← solution moindres carrés pour tout A de plein rang colonne
```

---

## 🪤 Piège 7 — Matrice mal conditionnée

Une matrice peut être inversible mais **quasi-singulière** :

```
κ(A) = σ_max / σ_min ≫ 1

Si κ(A) = 10⁶, une erreur relative de 10⁻¹⁰ dans b
peut engendrer une erreur relative de 10⁻⁴ dans x.
```

Symptôme : de légères variations dans les données entraînent des variations massives dans la solution.
Remède : régularisation (ajout de λI), ou SVD tronquée.

---

## 🪤 Piège 8 — Orthogonalité ≠ orthonormalité

```
Orthogonaux   : uᵢ · uⱼ = 0   pour i ≠ j   (perpendiculaires)
Orthonormaux  : uᵢ · uⱼ = 0   pour i ≠ j
              ET  ‖uᵢ‖ = 1

Matrice orthogonale au sens matriciel : QᵀQ = I   →   colonnes orthonormées
```

Une matrice à colonnes orthogonales (non normalisées) vérifie QᵀQ = D (diagonale), pas I.

---

## Récapitulatif rapide

| Piège | À retenir |
|-------|-----------|
| AB ≠ BA | L'ordre du produit compte toujours |
| det = 0 ≠ matrice nulle | Dépendance linéaire, pas nullité |
| Matrice carrée ≠ inversible | Vérifier det ≠ 0 |
| λ vs v | Valeur propre (scalaire) ≠ vecteur propre |
| Dimensions incompatibles | Dimension intérieure (k) doit coïncider |
| x = A⁻¹b | Valide seulement si A carrée et inversible |
| κ(A) ≫ 1 | Inversion numériquement instable |
| Orthogonal ≠ orthonormal | Orthonormal impose aussi ‖v‖ = 1 |
