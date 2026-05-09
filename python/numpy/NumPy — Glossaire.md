#numpy #glossaire #référence

| Terme                | Définition                                                                                                                             |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **ndarray**          | Objet tableau N-dimensionnel de NumPy (`numpy.ndarray`). La structure de données centrale de NumPy.                                    |
| **ndim**             | Nombre de dimensions d'un array. 1 = vecteur, 2 = matrice, 3+ = tenseur.                                                               |
| **shape**            | Tuple décrivant la taille de chaque dimension. Ex : `(3, 4)` = 3 lignes, 4 colonnes.                                                   |
| **size**             | Nombre total d'éléments dans l'array. Toujours égal au produit des valeurs de `shape`.                                                 |
| **dtype**            | Type de données homogène de tous les éléments. Ex : `float64`, `int32`, `bool`.                                                        |
| **axis**             | Dimension le long de laquelle une opération est effectuée. `axis=0` = lignes, `axis=1` = colonnes.                                     |
| **broadcasting**     | Mécanisme permettant d'effectuer des opérations entre arrays de shapes différentes en "étirant" le plus petit.                         |
| **ufunc**            | Fonction universelle (Universal Function) — opère élément par élément de façon vectorisée et optimisée en C. Ex : `np.sqrt`, `np.exp`. |
| **vue (view)**       | Array qui partage les mêmes données en mémoire qu'un autre array. Modifier la vue modifie l'original.                                  |
| **copie (copy)**     | Array indépendant avec ses propres données en mémoire. Modifier la copie ne modifie pas l'original.                                    |
| **slice**            | Extraction d'une portion d'array via `[start:stop:step]`. Retourne toujours une vue.                                                   |
| **fancy indexing**   | Indexation via une liste ou un array d'indices. Retourne toujours une copie.                                                           |
| **masque booléen**   | Array de `True/False` utilisé pour filtrer un array. Ex : `a[a > 4]`.                                                                  |
| **upcasting**        | Conversion automatique vers le type le plus général lors d'un mélange de types. Ex : int + float → float64.                            |
| **vectorisation**    | Remplacement de boucles Python par des opérations NumPy appliquées sur tout l'array d'un coup.                                         |
| **contiguïté**       | Un array est C-contiguous si ses données sont stockées en mémoire ligne par ligne — condition pour que reshape retourne une vue.       |
| **seed**             | Valeur initiale du générateur aléatoire garantissant des résultats reproductibles.                                                     |
| **in-place**         | Opération qui modifie l'array directement sans créer de nouvel objet. Ex : `a += 1`.                                                   |
| **produit scalaire** | Somme des produits des éléments correspondants de deux vecteurs. `np.dot(a, b)` ou `a @ b`.                                            |
| **norme**            | Longueur d'un vecteur. Norme euclidienne = `√(Σ xᵢ²)`. Calculée avec `np.linalg.norm`.                                                 |
| **tenseur**          | Array de dimension 3 ou plus. Utilisé notamment pour représenter des images (hauteur × largeur × canaux).                              |
| **SVD**              | Singular Value Decomposition — décomposition matricielle fondamentale utilisée en PCA et compression.                                  |