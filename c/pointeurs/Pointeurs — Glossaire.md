#c #pointeurs #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **pointeur** | Variable contenant une adresse mémoire. |
| **adresse** | Numéro identifiant un octet en mémoire. Obtenu avec `&var`. |
| **déréférencement** | Accès à la valeur à l'adresse stockée dans un pointeur. Opérateur `*`. |
| **pointeur nul** | Pointeur initialisé à `NULL` (adresse 0). Déréférencer un pointeur nul → SIGSEGV. |
| **pointeur non initialisé** | Pointeur dont la valeur est indéterminée. Déréférencer → UB. |
| **void*** | Pointeur générique sans type. Accepte tout pointeur de données. Ne peut pas être déréférencé sans cast. |
| **const T*** | Pointeur vers valeur constante. La valeur pointée est en lecture seule. |
| **T *const** | Pointeur constant. L'adresse ne peut pas changer. |
| **arithmétique de pointeur** | Addition/soustraction d'un entier à un pointeur. L'unité est `sizeof(type)`. |
| **decay** | Conversion implicite d'un tableau en pointeur vers son premier élément lors d'un passage en argument. |
| **double pointeur (T**)** | Pointeur vers pointeur. Permet de modifier un pointeur depuis une fonction. |
| **pointeur de fonction** | Variable contenant l'adresse d'une fonction. Syntaxe : `type (*nom)(params)`. |
| **callback** | Fonction passée en argument à une autre fonction via un pointeur de fonction. |
| **ptrdiff_t** | Type entier signé résultat de la soustraction de deux pointeurs. Défini dans `<stddef.h>`. |
| **undefined behavior (UB)** | Comportement non défini par le standard C. Résultat imprévisible : crash, valeur erronée, ou exécution silencieusement incorrecte. |
| **SIGSEGV** | Signal envoyé lors d'un accès mémoire invalide (segmentation fault). Cause fréquente : déréférencement de NULL ou pointeur invalide. |
| **aliasing** | Deux pointeurs de types différents pointant vers la même mémoire. Peut tromper l'optimiseur. Voir `restrict`. |
