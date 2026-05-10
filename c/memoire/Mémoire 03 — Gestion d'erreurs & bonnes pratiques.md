#c #memoire #bonnes-pratiques #erreurs #robustesse

## Toujours vérifier le retour de malloc

```c
// ❌ ignorer le NULL
int *p = malloc(1024);
p[0] = 42;   // crash si malloc a échoué

// ✅ vérifier systématiquement
int *p = malloc(1024);
if (!p) {
    perror("malloc");
    return NULL;   // ou exit(1) selon le contexte
}
```

## Macro d'allocation sécurisée

```c
#define MALLOC(type, n) ((type *)malloc_safe(sizeof(type) * (n)))

void *malloc_safe(size_t size) {
    void *p = malloc(size);
    if (!p) { perror("malloc"); exit(EXIT_FAILURE); }
    return p;
}

int *arr = MALLOC(int, 100);
```

## Pattern owner — qui libère quoi

```c
// Règle : celui qui alloue documente qui libère
// Conventions courantes :

// 1. L'appelant libère
char *strdup_manual(const char *s) {
    char *p = malloc(strlen(s) + 1);
    if (p) strcpy(p, s);
    return p;   // appelant doit free
}

// 2. La bibliothèque fournit une fonction de destruction
Buffer *buffer_new(size_t cap);
void    buffer_free(Buffer *b);   // symétrie explicite

// Ne jamais mélanger : allouer dans un contexte, free dans un autre sans convention claire
```

## Nullifier après free — éviter le dangling pointer

```c
free(p);
p = NULL;   // ✅ tout accès ultérieur via p → SIGSEGV détectable
            //    sans NULL → accès silencieux à mémoire libérée (UB)
```

## Macro free sécurisée

```c
#define SAFE_FREE(p) do { free(p); (p) = NULL; } while (0)

SAFE_FREE(arr);    // free + NULL en une fois
SAFE_FREE(NULL);   // no-op ✅
```

## Libérer dans l'ordre inverse de l'allocation

```c
// Structures imbriquées : libérer du plus profond au plus superficiel
typedef struct { char *name; int *data; } Record;

Record *r = malloc(sizeof *r);
r->name   = malloc(64);
r->data   = malloc(100 * sizeof(int));

// Libérer
free(r->data);   // 1. contenu
free(r->name);   // 2. contenu
free(r);         // 3. conteneur
```

## goto cleanup — gestion d'erreur propre

```c
int init(Context *ctx) {
    ctx->a = malloc(sizeof(A));
    if (!ctx->a) goto err_a;

    ctx->b = malloc(sizeof(B));
    if (!ctx->b) goto err_b;

    ctx->c = malloc(sizeof(C));
    if (!ctx->c) goto err_c;

    return 0;

err_c: free(ctx->b);
err_b: free(ctx->a);
err_a: return -1;
}
```

> [!tip] `goto cleanup` est idiomatique en C système
> C'est le pattern recommandé pour gérer plusieurs allocations qui peuvent échouer. Évite les ifs imbriqués et garantit la libération dans le bon ordre.

## Calculer la taille correctement

```c
// ✅ sizeof *ptr — immunisé contre les changements de type
int   *arr = malloc(n * sizeof *arr);
Point *pts = malloc(n * sizeof *pts);

// ❌ sizeof(int) — fragile si le type change
int *arr = malloc(n * sizeof(int));
```

> [!warning] Overflow dans le calcul de taille
> `n * sizeof(type)` peut déborder si `n` est grand. Utiliser `size_t` pour `n` et vérifier l'overflow ou utiliser `calloc(n, sizeof *p)` qui gère ce cas.
