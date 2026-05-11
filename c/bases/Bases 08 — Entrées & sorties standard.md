#c #bases #io #printf #scanf

## `printf` — afficher

```c
#include <stdio.h>

printf("Bonjour\n");                  // texte brut
printf("x = %d\n", x);               // entier
printf("f = %.2f\n", 3.14159);        // float/double, 2 décimales
printf("c = %c\n", 'A');             // caractère
printf("s = %s\n", "hello");         // chaîne
printf("p = %p\n", (void *)ptr);     // adresse (pointeur)
printf("n = %zu\n", sizeof(int));    // size_t
printf("n = %ld\n", (long)n);        // long
```

## Tableau de spécificateurs de format

| Spécificateur | Type | Exemple |
|---|---|---|
| `%d` | `int` | `42` |
| `%u` | `unsigned int` | `42` |
| `%ld` | `long` | `1234567890L` |
| `%lld` | `long long` | |
| `%f` | `float` / `double` | `3.140000` |
| `%e` | notation scientifique | `3.14e+00` |
| `%.2f` | 2 décimales | `3.14` |
| `%c` | `char` | `A` |
| `%s` | `char *` | `hello` |
| `%p` | pointeur | `0x7fff...` |
| `%zu` | `size_t` | `8` |
| `%x` | hexadécimal | `2a` |
| `%o` | octal | `52` |
| `%%` | littéral `%` | `%` |

## Largeur et alignement

```c
printf("%10d\n",  42);    // "        42" — aligné à droite sur 10
printf("%-10d|\n", 42);   // "42        |" — aligné à gauche
printf("%05d\n",  42);    // "00042" — rembourré de zéros
printf("%+d\n",   42);    // "+42" — force le signe
```

## `scanf` — lire depuis l'entrée standard

```c
int n;
scanf("%d", &n);        // ✅ lire un entier — passer l'adresse !

float f;
scanf("%f", &f);        // ✅ float (pas %lf)

double d;
scanf("%lf", &d);       // ✅ double avec scanf (contrairement à printf)

char s[64];
scanf("%63s", s);       // ✅ lire un mot (s'arrête à l'espace), au plus 63 chars
```

> [!warning] `scanf` sans `&`
> ```c
> scanf("%d", n);    // ❌ UB — n n'est pas une adresse
> scanf("%d", &n);   // ✅
> ```
> Exception : `char s[]` est déjà une adresse — `scanf("%s", s)` sans `&`.

> [!warning] `scanf("%s", s)` non borné
> Sans limite de taille, un input trop long déborde. Toujours écrire `%63s` pour un tableau de 64 caractères.

## `fgets` — lire une ligne (recommandé)

```c
char buf[128];
fgets(buf, sizeof(buf), stdin);   // lit au plus 127 chars + '\0'
// inclut le '\n' final si présent — le retirer si nécessaire :
buf[strcspn(buf, "\n")] = '\0';
```

`fgets` est plus sûr que `scanf("%s")` pour lire des lignes entières.

## Vider le buffer d'entrée

```c
// Après scanf, des caractères peuvent rester dans stdin (ex : le '\n')
// Vider le buffer avant un fgets :
int c;
while ((c = getchar()) != '\n' && c != EOF);
```

## `putchar` / `getchar`

```c
putchar('A');          // affiche un caractère
int c = getchar();     // lit un caractère (retourne int pour pouvoir tester EOF)

// Lire jusqu'à EOF
int c;
while ((c = getchar()) != EOF) {
    putchar(c);
}
```

## `fprintf` / `stderr`

```c
fprintf(stderr, "Erreur : fichier introuvable\n");   // ✅ messages d'erreur sur stderr
fprintf(stdout, "OK\n");                              // équivalent à printf
```

> [!tip] `printf` retourne le nombre de caractères écrits
> Rarement utile, mais permet de détecter des erreurs d'écriture en vérifiant la valeur de retour.
