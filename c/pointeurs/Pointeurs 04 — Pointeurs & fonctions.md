#c #pointeurs #fonctions #paramètres

## Passage par valeur vs par pointeur

```c
// Par valeur — copie locale, original inchangé
void f_val(int x) { x = 99; }

// Par pointeur — modifie l'original
void f_ptr(int *x) { *x = 99; }

int n = 42;
f_val(n);     // n = 42 (inchangé)
f_ptr(&n);    // n = 99 ✅
```

## Retourner plusieurs valeurs

```c
// C ne retourne qu'une valeur — utiliser des pointeurs pour "retourner" plusieurs résultats
void minmax(int *arr, int n, int *min, int *max) {
    *min = *max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] < *min) *min = arr[i];
        if (arr[i] > *max) *max = arr[i];
    }
}

int arr[] = {3, 1, 4, 1, 5};
int mn, mx;
minmax(arr, 5, &mn, &mx);   // mn=1, mx=5
```

## Passer un tableau à une fonction

```c
// Ces trois déclarations sont équivalentes pour le compilateur
void f(int *arr, size_t n);
void f(int arr[], size_t n);
void f(int arr[10], size_t n);   // le 10 est ignoré

// Dans tous les cas, arr est un pointeur — sizeof(arr) = 8
```

## Retourner un pointeur depuis une fonction

```c
// ✅ Retourner un pointeur vers un objet statique
const char *status(int code) {
    static char buf[32];
    snprintf(buf, sizeof(buf), "code=%d", code);
    return buf;   // static → vit toute la durée du programme
}

// ✅ Retourner un pointeur vers heap alloué par la fonction
int *make_array(size_t n) {
    return malloc(n * sizeof(int));   // appelant responsable du free
}

// ❌ Retourner l'adresse d'une variable locale
int *bad(void) {
    int x = 42;
    return &x;   // UB — x détruit à la fin de la fonction
}
```

## Passage d'un double pointeur — modifier un pointeur

```c
// Pour modifier le pointeur lui-même (pas la valeur pointée)
void allocate(int **ptr, size_t n) {
    *ptr = malloc(n * sizeof(int));
}

int *arr = NULL;
allocate(&arr, 10);   // arr pointe maintenant vers le tableau alloué
free(arr);
```

## Pointeur sur struct — passage efficace

```c
typedef struct { double x, y, z; } Vec3;

// ❌ copie de 24 octets à chaque appel
double norm_val(Vec3 v) { return sqrt(v.x*v.x + v.y*v.y + v.z*v.z); }

// ✅ passage par pointeur — 8 octets (adresse)
double norm_ptr(const Vec3 *v) { return sqrt(v->x*v->x + v->y*v->y + v->z*v->z); }
```

> [!tip] `const T *` vs `T *const`
> `const T *ptr` : la valeur pointée est constante (lecture seule) — le pointeur peut changer.
> `T *const ptr` : le pointeur est constant — la valeur peut changer.
> Utiliser `const T *` pour les paramètres en lecture seule : documente l'intention et permet au compilateur de valider.
