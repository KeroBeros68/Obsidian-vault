#c #bases #tableaux #chaînes

## Tableaux statiques

```c
int tab[5];                         // déclaration — 5 int non initialisés ⚠️
int tab[5] = {1, 2, 3, 4, 5};      // déclaration + initialisation
int tab[]  = {1, 2, 3, 4, 5};      // taille déduite automatiquement (5)
int tab[5] = {0};                   // tous à 0 ✅ (seul le premier suffit)
```

## Accès aux éléments

```c
int tab[] = {10, 20, 30, 40, 50};

tab[0]   // 10 — premier élément (indices commencent à 0)
tab[4]   // 50 — dernier élément
tab[5]   // ❌ hors limites — UB, pas d'erreur à la compilation

int n = sizeof(tab) / sizeof(tab[0]);   // nombre d'éléments ✅
```

## Parcourir un tableau

```c
int tab[] = {1, 2, 3, 4, 5};
int n = sizeof(tab) / sizeof(tab[0]);

for (int i = 0; i < n; i++) {
    printf("%d ", tab[i]);
}
```

## Tableaux multidimensionnels

```c
int mat[3][4];                    // 3 lignes, 4 colonnes
int mat[2][3] = {{1,2,3},{4,5,6}};

mat[0][0]   // 1 (ligne 0, colonne 0)
mat[1][2]   // 6 (ligne 1, colonne 2)

for (int i = 0; i < 2; i++)
    for (int j = 0; j < 3; j++)
        printf("%d ", mat[i][j]);
```

Stockés en mémoire en ligne : `mat[0][0]`, `mat[0][1]`, ..., `mat[0][3]`, `mat[1][0]`...

## Chaînes de caractères

Une chaîne est un tableau de `char` terminé par `'\0'` (caractère nul, valeur 0).

```c
char s[] = "hello";      // {'h','e','l','l','o','\0'} — 6 octets
char *s  = "hello";      // pointeur vers littéral en .rodata — ⚠️ non modifiable

s[0] = 'H';              // ✅ si déclaré avec [] (tableau)
                         // ❌ UB si déclaré avec * (pointeur vers littéral)
```

```
s : [ h | e | l | l | o | \0 ]
      0   1   2   3   4   5
```

## Fonctions de `<string.h>`

```c
#include <string.h>

strlen(s)                     // longueur sans le '\0'
strcpy(dest, src)             // copie src dans dest ⚠️ pas de vérification de taille
strncpy(dest, src, n)         // copie au plus n caractères
strcat(dest, src)             // concatène src à la fin de dest ⚠️
strncat(dest, src, n)         // concatène au plus n caractères
strcmp(a, b)                  // 0 si égaux, <0 si a<b, >0 si a>b
strncmp(a, b, n)              // compare au plus n caractères
strchr(s, c)                  // pointeur vers première occurrence de c, ou NULL
strstr(s, sub)                // pointeur vers première occurrence de sub, ou NULL
```

> [!warning] `strcpy` et `strcat` ne vérifient pas la taille
> Si `dest` est trop petit, débordement de tampon (buffer overflow). Préférer `strncpy` / `strncat` ou `snprintf`.

```c
// Copie sécurisée avec snprintf
char dest[32];
snprintf(dest, sizeof(dest), "%s", src);   // ✅ garantit la terminaison '\0'
```

## Lire et comparer une chaîne

```c
char s[] = "bonjour";

printf("%s\n", s);            // afficher
printf("%zu\n", strlen(s));   // longueur = 7

// Comparer — jamais avec ==
if (strcmp(s, "bonjour") == 0) {   // ✅
    printf("égal\n");
}
if (s == "bonjour") { ... }        // ❌ compare les adresses, pas le contenu
```

> [!tip] `sizeof` vs `strlen`
> `sizeof("hello")` → 6 (inclut le `'\0'`).
> `strlen("hello")` → 5 (exclut le `'\0'`).
> `sizeof` est calculé à la compilation, `strlen` parcourt la mémoire à l'exécution.
