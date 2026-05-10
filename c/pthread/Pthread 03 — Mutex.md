#c #pthread #mutex #synchronisation

## Le problème — race condition

`counter++` n'est pas atomique. C'est 3 instructions assembleur :

```
LOAD  counter → registre
ADD   registre, 1
STORE registre → counter
```

Deux threads peuvent interleaver ces instructions → résultat final imprévisible.

```c
int counter = 0;

void *increment(void *arg) {
    for (int i = 0; i < 1000000; i++)
        counter++;    // ❌ race condition — résultat < 2 000 000
    return NULL;
}
```

## pthread_mutex_t

```c
#include <pthread.h>

// Initialisation statique
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

// Initialisation dynamique (nécessite destroy)
pthread_mutex_init(&mtx, NULL);
pthread_mutex_destroy(&mtx);
```

```c
pthread_mutex_lock(&mtx);     // bloque si déjà verrouillé par un autre thread
// --- section critique ---
counter++;
// --- fin section critique ---
pthread_mutex_unlock(&mtx);
```

## Correction de l'exemple

```c
int counter = 0;
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

void *increment(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&mtx);
        counter++;                    // ✅ un seul thread à la fois
        pthread_mutex_unlock(&mtx);
    }
    return NULL;
}
```

## pthread_mutex_trylock

```c
int ret = pthread_mutex_trylock(&mtx);
if (ret == 0) {
    // verrou acquis
    pthread_mutex_unlock(&mtx);
} else {
    // EBUSY — mutex déjà pris, on ne bloque pas
}
```

| Fonction | Comportement si mutex pris |
|----------|---------------------------|
| `pthread_mutex_lock` | Bloque jusqu'à disponibilité |
| `pthread_mutex_trylock` | Retourne `EBUSY` immédiatement |

## Deadlock

```c
// Thread 1          // Thread 2
lock(A);             lock(B);
lock(B);  // bloque  lock(A);  // bloque → deadlock infini
```

> [!warning] Deadlock
> Deux threads s'attendent mutuellement → blocage infini, aucune sortie possible.

> [!tip] Prévention
> Toujours acquérir les mutex dans le **même ordre** dans tous les threads. Si tous acquièrent A puis B, le deadlock est impossible.
