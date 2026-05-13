#algo #tri #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Tri par comparaisons** | Algorithme déterminant l'ordre uniquement via `a < b`. Borne inférieure : Ω(n log n). |
| **Tri linéaire** | Tri ne reposant pas sur des comparaisons. Peut atteindre O(n). Ex : counting sort, radix sort. |
| **Stable** | Un tri est stable si deux éléments égaux conservent leur ordre relatif d'origine. |
| **En place (in-place)** | Tri utilisant O(1) espace auxiliaire (hors pile de récursion). Ex : heap sort, quick sort. |
| **Diviser pour régner** | Paradigme : diviser le problème en sous-problèmes, résoudre récursivement, combiner. Ex : merge sort, quick sort. |
| **Best case** | Complexité sur l'entrée la plus favorable (ex : tableau déjà trié). |
| **Average case** | Complexité en espérance sur une entrée aléatoire uniforme. |
| **Worst case** | Complexité sur l'entrée la plus défavorable. Garantie absolue. |
| **Pivot** | Élément de référence dans quick sort autour duquel on partitionne le tableau. |
| **Partition** | Réorganisation du tableau en éléments ≤ pivot à gauche, > pivot à droite. |
| **Partition de Lomuto** | Partition simple : pivot = dernier élément, un seul pointeur i. O(n) comparaisons. |
| **Partition de Hoare** | Partition originale de Quicksort : deux pointeurs se rapprochant. Moins de swaps. |
| **Médiane de trois** | Choix du pivot = médiane de arr[low], arr[mid], arr[high]. Réduit le risque de worst case. |
| **Introsort** | Hybride Quick sort + Heap sort + Insertion sort. Utilisé par C++ `std::sort`. |
| **Timsort** | Hybride Merge sort + Insertion sort détectant les runs. Utilisé par Python `sorted()`. |
| **Tas max** | Arbre binaire complet où chaque parent ≥ ses enfants. La racine est le maximum. |
| **Heapify (sift-down)** | Opération restaurant la propriété de tas depuis un nœud vers ses descendants. O(log n). |
| **Build heap** | Construction d'un tas à partir d'un tableau non trié. O(n) (non O(n log n)). |
| **Counting sort** | Tri comptant les occurrences de chaque valeur. O(n + k), k = étendue des valeurs. |
| **Radix sort** | Tri chiffre par chiffre via counting sort stable. O(d · (n + b)). |
| **LSD (Least Significant Digit)** | Variante radix traitant du chiffre le moins significatif au plus significatif. |
| **MSD (Most Significant Digit)** | Variante radix traitant du chiffre de tête. Naturel pour les chaînes. |
| **Base (radix)** | Nombre de valeurs distinctes d'un "chiffre". Base 10 → 0..9 ; base 256 → 0..255. |
| **Récurrence de Merge Sort** | T(n) = 2·T(n/2) + O(n) → O(n log n) par le théorème maître. |
| **Récurrence de Quick Sort** | Average : T(n) = 2·T(n/2) + O(n) → O(n log n). Worst : T(n) = T(n-1) + O(n) → O(n²). |
| **Théorème maître** | Outil pour résoudre T(n) = a·T(n/b) + f(n). Cas 2 : a=b^c et f(n)=O(n^c) → T(n)=O(n^c log n). |
| **Borne inférieure Ω(n log n)** | Tout tri par comparaisons nécessite au moins n·log₂(n) comparaisons dans le pire cas. Prouvé par arbre de décision. |
| **Arbre de décision** | Arbre binaire modélisant toutes les exécutions possibles d'un tri par comparaisons. Hauteur ≥ log₂(n!). |
| **Inversion** | Paire (i,j) avec i<j et arr[i]>arr[j]. Le nb d'inversions = nb de décalages d'insertion sort. |
| **Run** | Sous-séquence déjà triée (croissante ou décroissante) dans le tableau. Exploitée par Timsort. |
| **Tortue** | Petit élément en fin de tableau dans bubble sort, remontant lentement (1 position/passe). |
| **Lapin** | Grand élément en début de tableau dans bubble sort, descendant rapidement. |
| **Galloping** | Mode de fusion de Timsort : recherche exponentielle quand un run domine l'autre. |
| **MIN_RUN** | Taille minimale d'un run dans Timsort (32–64). Les runs trop courts sont complétés par insertion sort. |
| **Gap (Shell sort)** | Distance entre éléments comparés. Décroît jusqu'à 1 (= insertion sort classique). |
| **Séquence de Knuth** | Gaps 1, 4, 13, 40, 121... (h = 3h+1) pour Shell sort. Donne O(n^1.5) worst case. |
| **Séquence de Ciura** | Gaps empiriques 1,4,10,23,57,132,301,701 pour Shell sort. Meilleurs résultats en pratique. |
| **Seau (bucket)** | Intervalle de valeurs dans bucket sort. Les éléments d'un seau sont triés indépendamment. |
| **Distribution uniforme** | Hypothèse de bucket sort : les valeurs sont réparties uniformément sur la plage. |
| **Invariant de boucle** | Propriété vraie avant et après chaque itération. Ex : après k passes de bubble sort, les k plus grands sont en place. |
| **Tri en ligne (online)** | Tri capable de traiter les éléments un par un à leur arrivée. Insertion sort est online. Merge/Quick sort ne le sont pas. |
| **Tri adaptatif** | Tri exploitant l'ordre déjà présent dans les données. Timsort, insertion sort, cocktail sort sont adaptatifs. |
| **Binary Insertion Sort** | Variante d'insertion sort utilisant bisect pour trouver la position en O(log n) comparaisons. Décalages toujours O(n²). |
| **Introsort** | Hybride Quick sort + Heap sort + Insertion sort. C++ std::sort. Pire cas O(n log n) garanti. |
| **Odd-even sort** | Variante de bubble sort triant alternativement les paires (0,1),(2,3),... puis (1,2),(3,4),... Parallélisable. |
| **Comb sort** | Bubble sort avec gap décroissant (facteur ~1.3). Élimine les tortues. O(n log n) average. |
