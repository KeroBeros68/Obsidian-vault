#math #algebre-lineaire #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **vecteur** | Liste ordonnée de n nombres. Représente une direction et une magnitude dans ℝⁿ. |
| **norme** | Longueur d'un vecteur. Norme L2 : ‖v‖ = √(Σvᵢ²). |
| **vecteur unitaire** | Vecteur de norme 1. Obtenu par normalisation : v̂ = v / ‖v‖. |
| **produit scalaire** | u · v = Σuᵢvᵢ = ‖u‖‖v‖cos(θ). Mesure l'alignement entre deux vecteurs. |
| **produit vectoriel** | u × v — vecteur perpendiculaire à u et v. Défini en ℝ³ uniquement. |
| **orthogonalité** | u · v = 0. Les vecteurs sont perpendiculaires. |
| **orthonormalité** | Ensemble de vecteurs orthogonaux ET de norme 1. |
| **base** | Ensemble de vecteurs linéairement indépendants engendrant l'espace. |
| **dimension** | Nombre de vecteurs dans une base. |
| **matrice** | Tableau rectangulaire m×n de réels. Représente une transformation linéaire ℝⁿ → ℝᵐ. |
| **transposée** | Aᵀ : échange lignes et colonnes. (AB)ᵀ = BᵀAᵀ. |
| **matrice identité** | Iₙ : diagonale = 1, reste = 0. Élément neutre du produit : AI = IA = A. |
| **matrice symétrique** | A = Aᵀ. Valeurs propres réelles, vecteurs propres orthogonaux. |
| **matrice orthogonale** | QᵀQ = I. Colonnes orthonormées. Préserve normes et angles. det = ±1. |
| **produit de Hadamard** | A ⊙ B : produit élément par élément. Distinct du produit matriciel. |
| **rang** | Nombre de lignes (ou colonnes) linéairement indépendantes. |
| **trace** | Somme des éléments diagonaux. tr(A) = Σaᵢᵢ = Σλᵢ. |
| **déterminant** | Scalaire mesurant le facteur d'expansion volumique de A. det = 0 → singulière. |
| **matrice inversible** | det(A) ≠ 0. A⁻¹ existe telle que AA⁻¹ = I. |
| **pseudo-inverse** | A⁺ : généralise l'inverse aux matrices non carrées ou singulières. |
| **système linéaire** | Ax = b. Solution unique si det(A) ≠ 0. |
| **élimination de Gauss** | Méthode de résolution par réduction triangulaire via opérations sur les lignes. |
| **moindres carrés** | Solution x* minimisant ‖Ax - b‖². Pour systèmes sur-déterminés. x* = (AᵀA)⁻¹Aᵀb. |
| **espace colonne** | Ensemble de tous les vecteurs Ax (combinaisons linéaires des colonnes de A). |
| **noyau (kernel)** | Ensemble des x tels que Ax = 0. dim(noyau) = n - rang(A). |
| **valeur propre (λ)** | Scalaire tel que Av = λv pour un vecteur propre v ≠ 0. |
| **vecteur propre** | Vecteur v ≠ 0 dont la direction est préservée par A : Av = λv. |
| **polynôme caractéristique** | det(A - λI) = 0. Équation dont les racines sont les valeurs propres. |
| **diagonalisation** | A = PDP⁻¹. P = vecteurs propres, D = diagonale des valeurs propres. |
| **décomposition spectrale** | A = QΛQᵀ pour A symétrique. Q orthogonale, Λ diagonale des valeurs propres. |
| **valeurs singulières** | σᵢ = √(valeurs propres de AᵀA). Généralisent les valeurs propres à toutes les matrices. |
| **SVD** | A = UΣVᵀ. Décomposition universelle : rotation × mise à l'échelle × rotation. |
| **décomposition LU** | PA = LU. L triangulaire inférieure, U triangulaire supérieure. |
| **décomposition QR** | A = QR. Q orthogonale, R triangulaire supérieure. |
| **décomposition de Cholesky** | A = LLᵀ pour matrices SDP. L triangulaire inférieure à diagonale positive. |
| **nombre de condition** | κ(A) = σ_max / σ_min. Mesure la sensibilité aux perturbations. |
| **définie positive (SDP)** | Matrice symétrique avec toutes les valeurs propres > 0. Pour tout x ≠ 0 : xᵀAx > 0. |
