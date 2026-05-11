#c #bases #types #variables

## Types primitifs

| Type | Taille (64 bits) | Plage indicative |
|------|-----------------|-----------------|
| `char` | 1 octet | -128 à 127 (ou 0 à 255) |
| `unsigned char` | 1 octet | 0 à 255 |
| `int` | 4 octets | -2 147 483 648 à 2 147 483 647 |
| `unsigned int` | 4 octets | 0 à 4 294 967 295 |
| `long` | 8 octets (Linux) | ±9.2 × 10¹⁸ |
| `float` | 4 octets | ~7 chiffres significatifs |
| `double` | 8 octets | ~15 chiffres significatifs |
| `size_t` | 8 octets | 0 à SIZE_MAX — pour les tailles |

> [!info] Tailles garanties par `<stdint.h>`
> Préférer `int32_t`, `uint8_t`, `int64_t`... pour du code portable.

## Déclaration et initialisation

```c
int x;          // déclaration — valeur indéterminée ⚠️
int y = 10;     // déclaration + initialisation ✅
int a, b, c;    // plusieurs variables du même type

float f = 3.14f;   // suffixe f pour float (sans f → double)
double d = 3.14;
char c = 'A';      // caractère entre apostrophes
```

> [!warning] Variable locale non initialisée
> Sa valeur est indéterminée — pas zéro. Lire une variable non initialisée est UB.

## Portée des variables

```c
int globale = 0;        // globale — visible dans tout le fichier

void f(void) {
    int locale = 1;     // locale — détruite à la sortie de f
    {
        int bloc = 2;   // visible uniquement dans ce bloc
    }
    // bloc inaccessible ici
}
```

## Modificateurs

```c
static int compteur = 0;    // locale static — persiste entre les appels
extern int variable;        // déclaration d'une variable définie ailleurs

const int MAX = 100;        // constante — valeur non modifiable
const char *MSG = "hello";  // pointeur vers chaîne constante
```

## Constantes

```c
#define TAILLE 256       // constante préprocesseur — sans type, sans portée
const int LIMITE = 256;  // constante typée — préférable dans le code C moderne

// Littéraux numériques
42        // entier décimal (int)
042       // entier octal (commence par 0)
0x2A      // entier hexadécimal
42L       // long
42U       // unsigned int
42UL      // unsigned long
3.14f     // float
3.14      // double
```

## `sizeof`

```c
sizeof(int)       // taille en octets du type
sizeof(x)         // taille en octets de la variable x
sizeof("hello")   // 6 : 5 caractères + '\0'
```

`sizeof` retourne un `size_t`. Utiliser `%zu` dans `printf`.

```c
printf("int: %zu octets\n", sizeof(int));   // ✅
```

## Conversion de types

```c
int i = 3;
double d = i;        // conversion implicite int → double ✅

double pi = 3.14;
int n = (int)pi;     // cast explicite → n = 3 (troncature, pas arrondi)

int a = 5, b = 2;
double r = (double)a / b;   // ✅ 2.5 — caster avant la division
double r2 = a / b;          // ❌ 2.0 — division entière d'abord
```

> [!tip] Règle de promotion
> Dans une expression mixte, le type le moins précis est promu vers le plus précis : `char` → `int` → `long` → `float` → `double`.
