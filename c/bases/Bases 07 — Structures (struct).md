#c #bases #struct #structures

## Déclarer et utiliser une struct

```c
struct Point {
    int x;
    int y;
};

struct Point p;       // déclaration d'une variable
p.x = 3;
p.y = 7;

struct Point q = {3, 7};    // initialisation positionnelle
struct Point r = {.x = 3, .y = 7};  // initialisation par champ (C99) ✅
```

## `typedef` — alias de type

```c
typedef struct {
    int x;
    int y;
} Point;             // Point est maintenant utilisable directement

Point p = {3, 7};   // sans "struct" devant ✅
```

## Accès aux membres

```c
Point p = {1, 2};
Point *ptr = &p;

p.x        // accès direct via la variable
ptr->x     // accès via pointeur (équivalent à (*ptr).x)
(*ptr).y   // forme longue — peu utilisée
```

## Structs imbriquées

```c
typedef struct {
    float x, y;
} Vec2;

typedef struct {
    Vec2 position;
    Vec2 vitesse;
    float masse;
} Corps;

Corps c = {
    .position = {0.0f, 0.0f},
    .vitesse  = {1.0f, 2.0f},
    .masse    = 1.5f,
};

printf("%.1f\n", c.position.x);   // 0.0
```

## Structs et fonctions

Les structs sont passées par valeur (copie). Pour éviter la copie et permettre la modification, passer un pointeur.

```c
typedef struct { int x, y; } Point;

// Passage par valeur — reçoit une copie
void afficher(Point p) {
    printf("(%d, %d)\n", p.x, p.y);
}

// Passage par pointeur — peut modifier l'original
void deplacer(Point *p, int dx, int dy) {
    p->x += dx;
    p->y += dy;
}

Point p = {1, 2};
afficher(p);         // (1, 2)
deplacer(&p, 3, 4);
afficher(p);         // (4, 6) ✅
```

## Retourner une struct

```c
Point creer(int x, int y) {
    Point p = {x, y};
    return p;   // retourne une copie — valide ✅ (contrairement aux pointeurs locaux)
}

Point p = creer(5, 10);
```

## Tableau de structs

```c
Point pts[3] = {{0,0}, {1,2}, {3,4}};

for (int i = 0; i < 3; i++)
    printf("(%d,%d)\n", pts[i].x, pts[i].y);
```

## Alignement et padding

```c
typedef struct {
    char  a;   // 1 octet
    // 3 octets de padding (alignement de int = 4)
    int   b;   // 4 octets
    char  c;   // 1 octet
    // 3 octets de padding (pour aligner la struct à 4)
} Exemple;

printf("%zu\n", sizeof(Exemple));   // 12, pas 6
```

> [!info] `offsetof` pour inspecter le layout
> ```c
> #include <stddef.h>
> printf("%zu\n", offsetof(Exemple, b));   // 4
> ```

> [!tip] Réordonner les champs pour réduire le padding
> Regrouper les champs de même taille. Ex : tous les `int` ensemble, puis les `char`. Réduit la taille de la struct en mémoire.
