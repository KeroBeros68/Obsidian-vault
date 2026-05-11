#c #bases #compilation

## Structure minimale d'un programme C

```c
#include <stdio.h>      // inclusion d'un en-tête standard

int main(void) {
    printf("Hello\n");
    return 0;           // code de retour au système (0 = succès)
}
```

`main` est le point d'entrée. Le système l'appelle au démarrage du programme.

## Compiler avec gcc

```bash
gcc fichier.c -o mon_programme    # compile et produit l'exécutable
./mon_programme                   # exécuter
```

## Options courantes

| Option | Rôle |
|--------|------|
| `-Wall` | Active la plupart des avertissements |
| `-Wextra` | Avertissements supplémentaires |
| `-Werror` | Traite les avertissements comme des erreurs |
| `-g` | Inclut les infos de debug (gdb, valgrind) |
| `-O2` | Optimisation niveau 2 |
| `-std=c11` | Force le standard C11 |
| `-o nom` | Nomme l'exécutable produit |

```bash
gcc -Wall -Wextra -Werror -g -std=c11 fichier.c -o prog   # ✅ invocation recommandée
```

## Le préprocesseur

Exécuté avant la compilation. Traite les directives `#`.

```c
#include <stdio.h>       // fichier système (cherché dans /usr/include)
#include "mon_fichier.h" // fichier local (cherché dans le répertoire courant)

#define MAX 100          // constante texte — remplacée partout dans le source
#define CARRE(x) ((x)*(x))  // macro fonction — parenthéser toujours
```

> [!warning] Macro sans parenthèses
> ```c
> #define DOUBLE(x) x*2
> DOUBLE(3+1)   // → 3+1*2 = 5  ❌ pas 8
> #define DOUBLE(x) ((x)*2)   // ✅
> ```

## Compilation multi-fichiers

```
projet/
├── main.c
├── calcul.c
└── calcul.h
```

```c
// calcul.h — déclarations publiques
int addition(int a, int b);

// calcul.c — définitions
#include "calcul.h"
int addition(int a, int b) { return a + b; }

// main.c
#include "calcul.h"
int main(void) { printf("%d\n", addition(2, 3)); return 0; }
```

```bash
gcc -Wall -g main.c calcul.c -o projet   # compiler tous les .c ensemble
```

## Étapes de compilation

```
source.c → [préprocesseur] → source étendu
         → [compilateur]   → fichier objet (.o)
         → [éditeur de liens (ld)] → exécutable
```

> [!tip] Garder `-Wall -Wextra -Werror` en permanence
> Les avertissements signalent souvent de vrais bugs. Traiter chaque warning comme une erreur évite les surprises à l'exécution.
