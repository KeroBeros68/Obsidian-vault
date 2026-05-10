#c #memoire #pile #heap #durée-de-vie

## Les deux zones mémoire principales

```
Espace d'adressage d'un processus
┌─────────────────────┐ adresses hautes
│       Stack (pile)  │ ← grandit vers le bas
│          ↓          │    variables locales, paramètres, adresses de retour
├─────────────────────┤
│                     │ zone libre
├─────────────────────┤
│          ↑          │
│       Heap (tas)    │ ← grandit vers le haut
│                     │    malloc / calloc / realloc
├─────────────────────┤
│    BSS (.bss)       │    variables globales/statiques non initialisées
├─────────────────────┤
│    Data (.data)     │    variables globales/statiques initialisées
├─────────────────────┤
│    Code (.text)     │    instructions du programme
└─────────────────────┘ adresses basses
```

## Stack — pile d'exécution

```c
void f(void) {
    int x = 42;       // alloué sur la stack à l'entrée dans f
    char buf[256];    // 256 octets sur la stack
}                     // x et buf détruits ici automatiquement
```

| Caractéristique | Valeur |
|-----------------|--------|
| Allocation | Automatique (push/pop du frame) |
| Libération | Automatique à la sortie de fonction |
| Vitesse | Très rapide (un registre) |
| Taille | Limitée (~8 Mo par défaut sur Linux) |
| Durée de vie | Durée de la fonction / du bloc |

## Heap — mémoire dynamique

```c
int *p = malloc(sizeof(int) * 100);   // alloué sur le heap
// ... utiliser p ...
free(p);    // libération manuelle obligatoire
p = NULL;   // bonne pratique : annuler le pointeur après free
```

| Caractéristique | Valeur |
|-----------------|--------|
| Allocation | Manuelle (`malloc`, `calloc`, `realloc`) |
| Libération | Manuelle (`free`) |
| Vitesse | Plus lente (appel système potentiel) |
| Taille | Limitée par la RAM + swap |
| Durée de vie | Jusqu'au `free` ou fin du processus |

## Variables globales et statiques

```c
int global = 0;         // .data — durée de vie = programme entier

void f(void) {
    static int count = 0;   // .data — persiste entre les appels
    count++;
}
```

## Choisir entre stack et heap

```c
// Stack ✅ — taille connue, petite, durée de vie = fonction
int arr[64];

// Heap ✅ — taille inconnue à la compilation ou grande
int *arr = malloc(n * sizeof(int));

// Stack ❌ — trop grande → stack overflow
char buf[10 * 1024 * 1024];   // 10 Mo sur la stack → SIGSEGV

// Stack ❌ — retourner son adresse → UB
int *bad(void) { int x = 42; return &x; }
```

> [!warning] Stack overflow
> La stack est limitée. Une récursion profonde ou un grand tableau local peuvent la dépasser → SIGSEGV. Utiliser le heap pour les grandes allocations.

> [!tip] Règle de choix
> Si la taille est connue et petite (< quelques Ko) et que l'objet ne doit pas survivre à la fonction → stack.
> Sinon → heap.
