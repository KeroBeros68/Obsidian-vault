#c #pthread #condition #synchronisation

## Le problème — busy-wait

```c
// Thread consommateur — brûle du CPU inutilement ❌
while (queue_empty())
    ;
```

La variable de condition permet d'**attendre qu'une condition soit vraie** sans polling.

## pthread_cond_t

```c
pthread_cond_t  cond = PTHREAD_COND_INITIALIZER;
pthread_mutex_t mtx  = PTHREAD_MUTEX_INITIALIZER;

// Initialisation dynamique
pthread_cond_init(&cond, NULL);
pthread_cond_destroy(&cond);
```

## Pattern wait / signal

```c
// Consommateur — attendre
pthread_mutex_lock(&mtx);
while (!condition)                     // ⚠️ while obligatoire, jamais if
    pthread_cond_wait(&cond, &mtx);    // libère mtx, bloque, re-lock au réveil
// condition est vraie ici, mtx est détenu
pthread_mutex_unlock(&mtx);

// Producteur — signaler
pthread_mutex_lock(&mtx);
condition = 1;
pthread_cond_signal(&cond);            // réveille 1 thread en attente
pthread_mutex_unlock(&mtx);
```

> [!warning] Spurious wakeups — toujours `while`, jamais `if`
> `pthread_cond_wait` peut se réveiller sans que `signal` ait été appelé. C'est garanti par le standard POSIX. Avec `if`, le thread continue alors que la condition est encore fausse.

> [!tip] Ce que fait `pthread_cond_wait` atomiquement
> 1. Relâche le mutex
> 2. Met le thread en sommeil
> 3. Re-acquiert le mutex au réveil
>
> Appeler `wait` sans détenir le mutex est un undefined behavior.

## signal vs broadcast

```c
pthread_cond_signal(&cond);     // réveille 1 thread (le plus ancien en attente)
pthread_cond_broadcast(&cond);  // réveille TOUS les threads en attente
```

| | `signal` | `broadcast` |
|--|----------|-------------|
| Threads réveillés | 1 | Tous |
| Usage typique | File avec 1 item disponible | Changement d'état global, shutdown |

## Exemple complet — producteur/consommateur minimal

```c
int data  = 0;
int ready = 0;
pthread_mutex_t mtx  = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t  cond = PTHREAD_COND_INITIALIZER;

void *producer(void *arg) {
    pthread_mutex_lock(&mtx);
    data  = 42;
    ready = 1;
    pthread_cond_signal(&cond);
    pthread_mutex_unlock(&mtx);
    return NULL;
}

void *consumer(void *arg) {
    pthread_mutex_lock(&mtx);
    while (!ready)
        pthread_cond_wait(&cond, &mtx);
    printf("Reçu : %d\n", data);    // ✅ data protégé par mtx
    pthread_mutex_unlock(&mtx);
    return NULL;
}
```
