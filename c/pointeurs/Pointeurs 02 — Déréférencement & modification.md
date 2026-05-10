#c #pointeurs #déréférencement #modification

## Lire et modifier via un pointeur

```c
int x = 10;
int *p = &x;

printf("%d\n", *p);   // lire  → 10
*p = 99;              // écrire via le pointeur
printf("%d\n", x);    // x vaut maintenant 99 ✅
```

`*p = 99` modifie la variable `x` directement en mémoire.

## Passage par pointeur — simuler le passage par référence

En C, les arguments sont passés **par valeur**. Pour modifier une variable dans une fonction, on passe son adresse.

```c
void increment(int *n) {
    (*n)++;           // déréférencer puis incrémenter
}

int main(void) {
    int x = 5;
    increment(&x);    // passer l'adresse
    printf("%d\n", x);  // 6 ✅
}
```

```c
// ❌ passage par valeur — x non modifié
void increment_wrong(int n) {
    n++;   // modifie une copie locale
}
```

## Échanger deux valeurs — swap

```c
void swap(int *a, int *b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

int x = 1, y = 2;
swap(&x, &y);   // x=2, y=1
```

## Priorité des opérateurs avec *

```c
int *p = &x;

(*p)++;    // ✅ incrémenter la valeur pointée
*p++;      // ❌ incrémente le pointeur (pas la valeur) — équivalent à *(p++)
```

> [!warning] `*p++` vs `(*p)++`
> L'opérateur `++` post-fixe a une priorité plus haute que `*`. `*p++` déréférence p puis avance le pointeur. Toujours parenthéser : `(*p)++`.

## Pointeur sur struct — opérateur `->`

```c
typedef struct {
    int x;
    int y;
} Point;

Point pt = {1, 2};
Point *pp = &pt;

pp->x = 10;       // ✅ équivalent à (*pp).x = 10
printf("%d\n", (*pp).y);   // 2
```

`->` est du sucre syntaxique pour `(*ptr).membre`.

## Pointeur sur pointeur — double indirection

```c
int  x  = 42;
int *p  = &x;
int **pp = &p;

printf("%d\n", **pp);   // 42 — deux niveaux de déréférencement
*pp = NULL;             // modifie p (pas x)
```

> [!tip] Lecture d'une déclaration de pointeur
> Lire de droite à gauche depuis le nom de variable :
> `int **pp` → "pp est un pointeur vers un pointeur vers int"
> `char *argv[]` → "argv est un tableau de pointeurs vers char"
