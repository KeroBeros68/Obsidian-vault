#c #bases #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **type primitif** | Type de base fourni par le langage : `int`, `char`, `float`, `double`, `void`. |
| **variable locale** | Variable déclarée dans une fonction ou un bloc. Stockée sur la pile, détruite à la fin du bloc. |
| **variable globale** | Variable déclarée hors de toute fonction. Initialisée à 0, visible dans tout le fichier. |
| **`static` local** | Variable locale qui persiste entre les appels de la fonction. Initialisée une seule fois. |
| **`extern`** | Déclaration d'une variable ou fonction définie dans un autre fichier. |
| **`const`** | Qualificatif indiquant qu'une valeur ne doit pas être modifiée. |
| **`sizeof`** | Opérateur (pas une fonction) retournant la taille en octets d'un type ou d'une variable. Évalué à la compilation. |
| **`size_t`** | Type entier non signé pour représenter des tailles. Retourné par `sizeof` et `strlen`. Format `%zu`. |
| **prototype** | Déclaration d'une fonction sans son corps. Permet d'appeler la fonction avant sa définition. |
| **passage par valeur** | Le paramètre reçoit une copie de l'argument. Les modifications ne se propagent pas à l'appelant. |
| **tableau statique** | Tableau de taille fixe, alloué à la compilation sur la pile (ou en segment de données si global). |
| **decay** | Conversion implicite d'un tableau en pointeur vers son premier élément lors d'un passage en argument. |
| **chaîne de caractères** | Tableau de `char` terminé par le caractère nul `'\0'`. |
| **`'\0'`** | Caractère nul (valeur 0). Marque la fin d'une chaîne C. Distinct de `'0'` (valeur 48). |
| **`struct`** | Type composite regroupant plusieurs champs de types potentiellement différents. |
| **`typedef`** | Crée un alias de type. `typedef struct {...} Nom;` permet d'utiliser `Nom` sans `struct`. |
| **padding** | Octets insérés par le compilateur entre les champs d'une struct pour respecter l'alignement. |
| **`printf`** | Fonction d'affichage formaté sur `stdout`. |
| **`scanf`** | Fonction de lecture formatée depuis `stdin`. Requiert des adresses (`&var`). |
| **`fgets`** | Lecture sécurisée d'une ligne depuis un flux. Préférable à `scanf("%s")`. |
| **`stderr`** | Flux de sortie standard pour les erreurs. Distinct de `stdout`, non bufférisé par défaut. |
| **UB (Undefined Behavior)** | Comportement non défini par le standard C. Résultat imprévisible : crash, donnée corrompue, ou exécution silencieuse erronée. |
| **préprocesseur** | Étape avant compilation traitant les directives `#include`, `#define`, `#ifdef`... |
| **fichier en-tête (.h)** | Fichier contenant des déclarations partagées (prototypes, types, macros). Inclus avec `#include`. |
| **code de retour** | Valeur retournée par `main` au système. `0` = succès, valeur non nulle = erreur. |
