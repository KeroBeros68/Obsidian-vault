#c #pointeurs #arithmétique #tableaux

## Arithmétique des pointeurs

Ajouter `n` à un pointeur avance de `n * sizeof(type)` octets.

```c
int arr[] = {10, 20, 30, 40};
int *p = arr;   // p pointe sur arr[0]

p + 1;    // adresse de arr[1] (+4 octets sur 64 bits)
p + 2;    // adresse de arr[2] (+8 octets)
*(p + 1); // valeur de arr[1] = 20
```

```
arr: [10][20][30][40]
      ↑    ↑    ↑    ↑
      p   p+1  p+2  p+3
```

## Tableau et pointeur — équivalence

Un tableau en C **est** un pointeur constant vers son premier élément.

```c
int arr[] = {1, 2, 3};

arr[i]    ≡  *(arr + i)    // notation équivalentes
&arr[i]   ≡  arr + i

arr[0]    ≡  *arr
arr[1]    ≡  *(arr + 1)
```

```c
// Parcourir avec arithmétique de pointeur
int *p = arr;
int *end = arr + 3;   // un passé le dernier élément

while (p < end) {
    printf("%d ", *p);
    p++;
}
```

## Différence entre deux pointeurs

```c
int arr[] = {10, 20, 30};
int *a = &arr[0];
int *b = &arr[2];

ptrdiff_t diff = b - a;   // 2 (en nombre d'éléments, pas d'octets)
```

`ptrdiff_t` (défini dans `<stddef.h>`) est le type résultat d'une soustraction de pointeurs.

## Tableau vs pointeur — différences

```c
int arr[5] = {0};
int *p = arr;

sizeof(arr);    // 20 octets (5 * 4) — taille du tableau entier ✅
sizeof(p);      // 8 octets — taille du pointeur ⚠️

arr = p;        // ❌ erreur de compilation — arr est constant
p = arr;        // ✅ p peut être réassigné
```

> [!warning] sizeof sur un pointeur ≠ taille du tableau
> Quand un tableau est passé à une fonction, il **decay** en pointeur. `sizeof` dans la fonction donne la taille du pointeur, pas du tableau. Toujours passer la taille explicitement.

```c
void print(int *arr, size_t n) {   // ✅ passer n
    for (size_t i = 0; i < n; i++)
        printf("%d\n", arr[i]);
}
```

## Tableaux 2D et pointeurs

```c
int mat[3][4];

// mat[i][j] ≡ *(*(mat + i) + j)
// mat[i]    est un tableau de 4 int (pas un pointeur simple)

// Pointeur vers une ligne
int (*row)[4] = mat;    // pointeur vers tableau de 4 int
row++;                  // avance de sizeof(int[4]) = 16 octets
```

> [!tip] Opérations autorisées sur les pointeurs
> ✅ `p + n`, `p - n`, `p++`, `p--`, `p - q` (même tableau), `p[i]`
> ❌ `p + q`, `p * n`, `p / n`, comparer des pointeurs vers des objets distincts
