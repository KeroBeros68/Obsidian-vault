#c #pointeurs #bases #adresses

## Adresse mémoire

Chaque variable occupe un emplacement en mémoire identifié par une **adresse**.

```c
int x = 42;
printf("%p\n", (void *)&x);   // ex: 0x7ffd3a2c1b4c
```

```
Mémoire
┌──────────┬──────────┐
│  adresse │  valeur  │
├──────────┼──────────┤
│ 0x...b4c │    42    │  ← variable x
└──────────┴──────────┘
```

## Déclarer un pointeur

```c
int  *p;      // pointeur vers int
char *c;      // pointeur vers char
double *d;    // pointeur vers double
```

Un pointeur est une variable qui **contient une adresse**.

## Opérateurs fondamentaux

```c
int x = 42;
int *p;

p = &x;       // & : adresse de x  →  p contient l'adresse de x
int v = *p;   // * : déréférencement → v = valeur pointée = 42
```

```
x  : [42]  @ adresse 0x100
p  : [0x100]               ← p stocke l'adresse de x
*p : [42]                  ← valeur à l'adresse stockée dans p
```

## Taille d'un pointeur

```c
sizeof(int *)    // 8 octets sur 64 bits, 4 sur 32 bits
sizeof(char *)   // identique — tous les pointeurs ont la même taille
sizeof(void *)   // identique
```

La taille d'un pointeur est indépendante du type pointé. Elle dépend de l'architecture.

## Initialisation — pointeur nul

```c
int *p = NULL;   // pointeur nul — ne pointe nulle part ✅

// Tester avant de déréférencer
if (p != NULL) {
    int v = *p;
}
```

> [!warning] Pointeur non initialisé — undefined behavior
> ```c
> int *p;      // valeur indéterminée — pointe n'importe où
> *p = 42;     // ❌ UB — crash probable (SIGSEGV) ou corruption silencieuse
> ```
> Toujours initialiser à `NULL` ou à une adresse valide.

## Résumé des opérateurs

| Opérateur | Nom | Rôle |
|-----------|-----|------|
| `&var` | Adresse-de | Donne l'adresse de `var` |
| `*ptr` | Déréférencement | Accède à la valeur pointée |
| `ptr` | (valeur du pointeur) | L'adresse elle-même |
