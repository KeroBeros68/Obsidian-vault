#c #pointeurs #fonctions #callbacks #avancé

## Déclarer un pointeur de fonction

```c
// Syntaxe : type_retour (*nom)(types_paramètres)
int (*fp)(int, int);   // pointeur vers fonction prenant 2 int, retournant int

// Assigner
int add(int a, int b) { return a + b; }
fp = add;              // ✅ sans & (le nom de fonction est déjà une adresse)
fp = &add;             // ✅ équivalent
```

## Appeler via un pointeur de fonction

```c
int result = fp(3, 4);    // ✅ appel direct
int result = (*fp)(3, 4); // ✅ équivalent explicite
```

## typedef — simplifier la déclaration

```c
typedef int (*BinOp)(int, int);   // BinOp = type "pointeur vers int(int,int)"

int add(int a, int b) { return a + b; }
int mul(int a, int b) { return a * b; }

BinOp op = add;
printf("%d\n", op(3, 4));   // 7
op = mul;
printf("%d\n", op(3, 4));   // 12
```

## Callbacks — passer une fonction en paramètre

```c
void apply(int *arr, size_t n, int (*transform)(int)) {
    for (size_t i = 0; i < n; i++)
        arr[i] = transform(arr[i]);
}

int double_it(int x) { return x * 2; }
int square(int x)    { return x * x; }

int arr[] = {1, 2, 3, 4};
apply(arr, 4, double_it);   // {2, 4, 6, 8}
apply(arr, 4, square);      // {4, 16, 36, 64}
```

## qsort — exemple standard

```c
#include <stdlib.h>

int cmp_int(const void *a, const void *b) {
    int x = *(const int *)a;
    int y = *(const int *)b;
    return (x > y) - (x < y);   // ✅ évite l'overflow de (x - y)
}

int arr[] = {5, 2, 8, 1, 9};
qsort(arr, 5, sizeof(int), cmp_int);
// arr = {1, 2, 5, 8, 9}
```

## Tableau de pointeurs de fonctions — table de dispatch

```c
typedef void (*Handler)(int);

void on_read(int fd)  { printf("read fd=%d\n",  fd); }
void on_write(int fd) { printf("write fd=%d\n", fd); }
void on_error(int fd) { printf("error fd=%d\n", fd); }

Handler dispatch[] = { on_read, on_write, on_error };

int event = 1;
dispatch[event](42);   // on_write(42)
```

## Retourner un pointeur de fonction

```c
typedef int (*Op)(int, int);

Op get_op(char c) {
    if (c == '+') return add;
    if (c == '*') return mul;
    return NULL;
}

Op f = get_op('+');
if (f) printf("%d\n", f(3, 4));
```

> [!tip] Lire une déclaration complexe
> `int (*fp)(int, int)` → "fp est un pointeur (`*fp`) vers une fonction prenant `(int, int)` et retournant `int`"
> Règle : commencer par le nom, aller à droite jusqu'à `)`, revenir à gauche.

> [!warning] Pointeur de fonction ≠ pointeur de données
> La conversion d'un pointeur de fonction en `void *` est un comportement indéfini en C strict (mais courant sur les plateformes POSIX via `dlsym`). Ne jamais caster arbitrairement.
