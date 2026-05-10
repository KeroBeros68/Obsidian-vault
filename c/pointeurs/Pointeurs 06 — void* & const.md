#c #pointeurs #void #const #généricité

## void* — pointeur générique

`void *` est un pointeur sans type. Il peut recevoir n'importe quel pointeur de données sans cast explicite en C.

```c
int    x = 42;
double d = 3.14;

void *p;
p = &x;   // ✅ pas de cast nécessaire en C
p = &d;   // ✅

// Déréférencer : cast obligatoire
int *ip = p;
printf("%d\n", *ip);   // 42
```

## Fonctions génériques avec void*

```c
// memcpy, memset, qsort, malloc retournent tous void*

void *memcpy(void *dest, const void *src, size_t n);
void *memset(void *s, int c, size_t n);
void *malloc(size_t size);

int arr[] = {1, 2, 3};
int cpy[3];
memcpy(cpy, arr, sizeof(arr));   // pas de cast nécessaire
```

## Règles du void*

```c
// ✅ Conversion implicite dans les deux sens (C uniquement, pas C++)
int *p = malloc(sizeof(int));   // malloc retourne void*, pas de cast

// ❌ Déréférencement interdit sans cast
void *v = &x;
*v = 10;          // erreur de compilation — type inconnu

// ❌ Arithmétique interdite
v + 1;            // erreur — sizeof(void) indéfini
```

## const — pointeur et constance

Quatre combinaisons possibles :

```c
int x = 1, y = 2;

int *p1 = &x;              // pointeur mutable, valeur mutable
*p1 = 99;  p1 = &y;       // ✅ les deux

const int *p2 = &x;        // pointeur mutable, valeur constante (lecture seule)
*p2 = 99;                  // ❌ erreur de compilation
p2 = &y;                   // ✅

int *const p3 = &x;        // pointeur constant, valeur mutable
*p3 = 99;                  // ✅
p3 = &y;                   // ❌ erreur de compilation

const int *const p4 = &x;  // pointeur constant, valeur constante
*p4 = 99;                  // ❌
p4 = &y;                   // ❌
```

## Lecture rapide — règle Est/Ouest

```
const int *p    →  "ce qui est à gauche de * est const"  →  valeur const
int *const p    →  "ce qui est à droite de * est const"  →  pointeur const
```

## Paramètres en lecture seule

```c
// Documenter qu'on ne modifie pas les données
size_t strlen_safe(const char *s) {
    size_t n = 0;
    while (*s++) n++;
    return n;
}

// const char * accepte char * et const char *
// char *       n'accepte pas const char *  ⚠️
```

## const et les chaînes littérales

```c
char *s1 = "hello";         // ❌ légal mais dangereux — "hello" est en .rodata
const char *s2 = "hello";   // ✅ correct — indique lecture seule

*s1 = 'H';    // UB — écriture dans .rodata → SIGSEGV
```

> [!tip] Règle pratique
> Utiliser `const T *` dès qu'une fonction ne modifie pas les données pointées. Le compilateur vérifie et documente l'intention.

> [!warning] Cast qui supprime const
> ```c
> void f(const int *p) {
>     int *q = (int *)p;   // ❌ légal à la compilation, UB si la cible est réellement const
>     *q = 0;
> }
> ```
> Ne jamais caster pour éliminer `const` sauf raison impérative documentée.
