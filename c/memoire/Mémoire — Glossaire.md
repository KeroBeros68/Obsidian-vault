#c #memoire #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **stack (pile)** | Zone mémoire pour les variables locales et frames de fonctions. Gestion automatique, taille limitée (~8 Mo). |
| **heap (tas)** | Zone mémoire pour les allocations dynamiques (`malloc`). Gestion manuelle, taille limitée par la RAM. |
| **frame de pile** | Bloc de stack alloué à l'entrée d'une fonction, contenant ses variables locales et l'adresse de retour. |
| **malloc** | Alloue `n` octets non initialisés sur le heap. Retourne `NULL` si échec. |
| **calloc** | Alloue `n * size` octets initialisés à 0. |
| **realloc** | Redimensionne un bloc existant. Peut déplacer le bloc. Retourne `NULL` si échec (bloc original intact). |
| **free** | Libère un bloc alloué. No-op sur `NULL`. Double free → UB. |
| **memory leak** | Mémoire allouée jamais libérée. S'accumule jusqu'à épuisement de la RAM. |
| **dangling pointer** | Pointeur vers de la mémoire libérée ou hors scope. Accès → UB. |
| **double free** | Appel de `free` deux fois sur le même pointeur → UB, souvent crash ou corruption. |
| **use-after-free** | Accès à de la mémoire libérée. UB silencieux ou crash. |
| **heap overflow** | Écriture au-delà d'un bloc alloué. Écrase d'autres données heap. |
| **stack overflow** | Dépassement de la capacité de la pile. Cause : récursion profonde ou grand tableau local. |
| **uninitialized read** | Lecture de mémoire allouée mais non initialisée (ex: via `malloc`). UB. |
| **fragmentation** | La mémoire heap se morcèle après de nombreux alloc/free. Ralentit et peut faire échouer de grandes allocations. |
| **Valgrind** | Outil d'analyse dynamique détectant les erreurs mémoire (fuites, use-after-free, etc.). Overhead ×10-50. |
| **AddressSanitizer (ASan)** | Sanitizer compilateur (`-fsanitize=address`) détectant les erreurs mémoire à l'exécution. Overhead ×2. |
| **ThreadSanitizer (TSan)** | Sanitizer détectant les race conditions (`-fsanitize=thread`). |
| **UBSan** | Sanitizer détectant l'undefined behavior (`-fsanitize=undefined`). |
| **goto cleanup** | Pattern idiomatique C pour libérer des ressources allouées en cas d'erreur dans une chaîne d'allocations. |
