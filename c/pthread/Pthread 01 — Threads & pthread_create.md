#c #pthread #création #bases

## Process vs Thread

Un processus possède son propre espace mémoire. Un thread vit à l'intérieur d'un processus et partage sa mémoire.

```
Processus
├── Code (.text)      ← partagé entre tous les threads
├── Données (.data)   ← partagé
├── Heap              ← partagé  ⚠️ source de race conditions
├── Thread 1
│   ├── Stack propre  ← privé
│   └── Registres     ← privé
└── Thread 2
    ├── Stack propre
    └── Registres
```

## pthread_create

```c
#include <pthread.h>

int pthread_create(
    pthread_t *thread,              // identifiant du thread créé (sortie)
    const pthread_attr_t *attr,     // attributs — NULL = défaut
    void *(*start_routine)(void *), // fonction à exécuter
    void *arg                       // argument passé à la fonction
);
// Retourne 0 si succès, code d'erreur sinon (pas -1)
```

| Paramètre | Rôle |
|-----------|------|
| `thread` | Reçoit l'identifiant du thread créé |
| `attr` | `NULL` pour les attributs par défaut |
| `start_routine` | Fonction à exécuter dans le thread |
| `arg` | Pointeur vers les données passées au thread |

## Signature obligatoire de la fonction thread

```c
void *ma_fonction(void *arg);
//  ^             ^
//  retourne void*  argument unique void* (cast nécessaire)
```

## Exemple minimal

```c
#include <pthread.h>
#include <stdio.h>

void *worker(void *arg) {
    int n = *(int *)arg;
    printf("Thread reçoit : %d\n", n);
    return NULL;
}

int main(void) {
    pthread_t tid;
    int val = 42;

    int err = pthread_create(&tid, NULL, worker, &val);
    if (err != 0) {
        fprintf(stderr, "pthread_create: %s\n", strerror(err));
        return 1;
    }
    pthread_join(tid, NULL);
    return 0;
}
```

## Passer plusieurs arguments — struct

```c
typedef struct {
    int x;
    int y;
} Args;

void *worker(void *arg) {
    Args *a = (Args *)arg;
    printf("%d + %d = %d\n", a->x, a->y, a->x + a->y);
    return NULL;
}

int main(void) {
    pthread_t tid;
    Args args = {10, 20};
    pthread_create(&tid, NULL, worker, &args);
    pthread_join(tid, NULL);
    return 0;
}
```

> [!tip] Compilation
> `gcc prog.c -o prog -lpthread`
> Le flag `-lpthread` est obligatoire pour lier la bibliothèque.

> [!warning] Variable locale passée en argument
> Ne jamais passer l'adresse d'une variable locale si la fonction appelante peut se terminer avant le thread. Le thread lirait de la mémoire libérée (undefined behavior).
