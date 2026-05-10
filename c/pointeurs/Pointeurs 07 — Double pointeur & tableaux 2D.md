#c #pointeurs #double-pointeur #tableaux-2D #avancé

## Double pointeur — int**

```c
int  x  = 42;
int *p  = &x;    // p  : adresse de x
int **pp = &p;   // pp : adresse de p

printf("%d\n",  x);    // 42
printf("%d\n", *p);    // 42
printf("%d\n", **pp);  // 42

**pp = 99;   // modifie x
*pp  = NULL; // modifie p (pas x)
```

```
pp → [ adresse de p ] → [ adresse de x ] → [ 42 ]
```

## Modifier un pointeur depuis une fonction

```c
void alloc(int **out, size_t n) {
    *out = malloc(n * sizeof(int));   // modifie le pointeur de l'appelant
}

int *arr = NULL;
alloc(&arr, 10);   // &arr est un int**
free(arr);
```

## Tableau de chaînes — char**

`char **` est le type de `argv` : tableau de pointeurs vers des chaînes.

```c
int main(int argc, char **argv) {
    // argv[0] = nom du programme
    // argv[1] ... argv[argc-1] = arguments
    // argv[argc] = NULL (sentinelle)

    for (int i = 0; i < argc; i++)
        printf("argv[%d] = %s\n", i, argv[i]);
}
```

```
argv → [ ptr → "prog\0" ]
       [ ptr → "-l\0"   ]
       [ ptr → "/tmp\0" ]
       [ NULL            ]
```

## Tableau 2D — trois représentations

### 1. Tableau statique (contigu en mémoire)

```c
int mat[3][4];             // 3 lignes, 4 colonnes — contigu
mat[i][j];                 // accès direct

void f(int mat[][4], int rows);   // le nombre de colonnes est obligatoire
```

### 2. Tableau de pointeurs (non contigu)

```c
int *rows[3];
for (int i = 0; i < 3; i++)
    rows[i] = malloc(4 * sizeof(int));

rows[i][j] = 42;          // accès via double indirection

// Libérer
for (int i = 0; i < 3; i++) free(rows[i]);
```

### 3. Tableau 1D simulant un 2D (contigu, taille dynamique)

```c
int rows = 3, cols = 4;
int *mat = malloc(rows * cols * sizeof(int));

mat[i * cols + j] = 42;   // accès : ligne * nb_colonnes + colonne ✅

free(mat);
```

## Comparaison

| | Statique `[M][N]` | Pointeurs `*rows[]` | 1D dynamique |
|--|-------------------|--------------------|----|
| Mémoire | Contiguë | Fragmentée | Contiguë |
| Taille | Compile-time | Runtime | Runtime |
| Cache | Optimal | Moins bon | Optimal |
| Libération | Automatique | Boucle de free | 1 free |

> [!tip] Préférer le tableau 1D dynamique
> `int *mat = malloc(rows * cols * sizeof(int))` avec accès `mat[i * cols + j]` : contigu, une seule allocation, cache-friendly — c'est ce que font numpy, BLAS, et la plupart des bibliothèques numériques.
