#c #memoire #malloc #calloc #realloc #free

## malloc — allouer sans initialiser

```c
#include <stdlib.h>

void *malloc(size_t size);
// Retourne un pointeur vers size octets non initialisés
// Retourne NULL si l'allocation échoue
```

```c
int *arr = malloc(10 * sizeof(int));
if (!arr) { perror("malloc"); exit(1); }
// contenu indéterminé — initialiser avant lecture
arr[0] = 42;
free(arr);
```

## calloc — allouer et zéroïser

```c
void *calloc(size_t nmemb, size_t size);
// Alloue nmemb * size octets, tous initialisés à 0
```

```c
int *arr = calloc(10, sizeof(int));
// arr[0] ... arr[9] valent 0 ✅
free(arr);
```

> [!tip] malloc vs calloc
> `calloc` est préférable quand on a besoin de zéros (tableaux, struct). `malloc` est légèrement plus rapide (pas de memset). Ne jamais lire la mémoire allouée par `malloc` sans l'initialiser.

## realloc — redimensionner une allocation

```c
void *realloc(void *ptr, size_t new_size);
// Peut déplacer le bloc en mémoire → ne jamais utiliser l'ancien pointeur après
// Retourne NULL si échec (bloc original intact)
```

```c
int *arr = malloc(10 * sizeof(int));

// ❌ Pattern dangereux — perte du pointeur si realloc échoue
arr = realloc(arr, 20 * sizeof(int));

// ✅ Pattern correct
int *tmp = realloc(arr, 20 * sizeof(int));
if (!tmp) {
    free(arr);   // libérer l'original
    return NULL;
}
arr = tmp;
```

## free — libérer la mémoire

```c
void free(void *ptr);
```

```c
free(ptr);      // libérer le bloc
ptr = NULL;     // ✅ éviter le dangling pointer

free(NULL);     // ✅ no-op — toujours sûr
free(ptr);
free(ptr);      // ❌ double free — UB
```

## Tableau récapitulatif

| Fonction | Initialisation | Paramètres | Usage typique |
|----------|---------------|------------|---------------|
| `malloc(n)` | Non | taille en octets | Allocation générale |
| `calloc(n, s)` | Zéros | nombre × taille | Tableau, struct initialisée |
| `realloc(p, n)` | Non (partie nouvelle) | pointeur + nouvelle taille | Tableau dynamique |
| `free(p)` | — | pointeur | Libérer toute allocation |

## Patterns courants

```c
// Allouer une struct
typedef struct { int x, y; } Point;
Point *p = malloc(sizeof *p);   // sizeof *p = sizeof(Point) ✅ plus sûr que sizeof(Point)
p->x = 1; p->y = 2;
free(p);

// Allouer un tableau de structs
Point *arr = calloc(100, sizeof *arr);
free(arr);

// Tableau dynamique (vecteur)
size_t cap = 8, len = 0;
int *vec = malloc(cap * sizeof *vec);
// ... push :
if (len == cap) {
    cap *= 2;
    int *tmp = realloc(vec, cap * sizeof *vec);
    if (!tmp) { free(vec); return NULL; }
    vec = tmp;
}
vec[len++] = value;
```

> [!warning] Jamais free une adresse non retournée par malloc/calloc/realloc
> ```c
> int arr[10];
> free(arr);   // ❌ UB — arr est sur la stack
> ```
