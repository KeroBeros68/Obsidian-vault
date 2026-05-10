#c #pthread #patterns #producteur-consommateur #thread-pool

## Producteur / Consommateur — file bornée

```c
#define CAPACITY 8

typedef struct {
    int      buf[CAPACITY];
    int      head, tail, count;
    pthread_mutex_t mtx;
    pthread_cond_t  not_full;   // signalé quand on peut produire
    pthread_cond_t  not_empty;  // signalé quand on peut consommer
} Queue;

void queue_push(Queue *q, int val) {
    pthread_mutex_lock(&q->mtx);
    while (q->count == CAPACITY)
        pthread_cond_wait(&q->not_full, &q->mtx);
    q->buf[q->tail] = val;
    q->tail = (q->tail + 1) % CAPACITY;
    q->count++;
    pthread_cond_signal(&q->not_empty);
    pthread_mutex_unlock(&q->mtx);
}

int queue_pop(Queue *q) {
    pthread_mutex_lock(&q->mtx);
    while (q->count == 0)
        pthread_cond_wait(&q->not_empty, &q->mtx);
    int val = q->buf[q->head];
    q->head = (q->head + 1) % CAPACITY;
    q->count--;
    pthread_cond_signal(&q->not_full);
    pthread_mutex_unlock(&q->mtx);
    return val;
}
```

> [!tip] Deux variables de condition
> `not_full` et `not_empty` séparent les causes de blocage. Utiliser une seule cond avec `broadcast` fonctionnerait mais réveillerait des threads inutilement.

## Thread pool — structure de base

```c
#define POOL_SIZE 4

typedef void (*Task)(void *);

typedef struct {
    Task     fn;
    void    *arg;
} Job;

typedef struct {
    pthread_t       workers[POOL_SIZE];
    Job             queue[64];
    int             head, tail, count;
    pthread_mutex_t mtx;
    pthread_cond_t  cond;
    int             stop;
} ThreadPool;

void *pool_worker(void *arg) {
    ThreadPool *pool = arg;
    while (1) {
        pthread_mutex_lock(&pool->mtx);
        while (pool->count == 0 && !pool->stop)
            pthread_cond_wait(&pool->cond, &pool->mtx);
        if (pool->stop && pool->count == 0) {
            pthread_mutex_unlock(&pool->mtx);
            return NULL;
        }
        Job job = pool->queue[pool->head];
        pool->head = (pool->head + 1) % 64;
        pool->count--;
        pthread_mutex_unlock(&pool->mtx);
        job.fn(job.arg);   // exécuter la tâche sans tenir le mutex
    }
}

void pool_submit(ThreadPool *pool, Task fn, void *arg) {
    pthread_mutex_lock(&pool->mtx);
    pool->queue[pool->tail] = (Job){fn, arg};
    pool->tail = (pool->tail + 1) % 64;
    pool->count++;
    pthread_cond_signal(&pool->cond);
    pthread_mutex_unlock(&pool->mtx);
}

void pool_shutdown(ThreadPool *pool) {
    pthread_mutex_lock(&pool->mtx);
    pool->stop = 1;
    pthread_cond_broadcast(&pool->cond);   // réveiller tous les workers
    pthread_mutex_unlock(&pool->mtx);
    for (int i = 0; i < POOL_SIZE; i++)
        pthread_join(pool->workers[i], NULL);
}
```

## Shutdown propre — flag volatile

```c
volatile int running = 1;   // volatile : empêche l'optimisation du compilateur

void *worker(void *arg) {
    while (running) {
        do_work();
    }
    return NULL;
}

// Depuis main ou un gestionnaire de signal
running = 0;
```

> [!warning] `volatile` n'est pas une garantie de thread-safety
> Il empêche le compilateur de cacher la variable en registre, mais ne garantit pas l'atomicité. Pour un arrêt fiable, protéger le flag avec un mutex ou utiliser `stdatomic.h` (`atomic_int`).
