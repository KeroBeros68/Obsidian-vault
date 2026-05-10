#c #pthread #spinlock #barrier #synchronisation

## Spinlock vs Mutex

| | Mutex | Spinlock |
|--|-------|----------|
| Thread bloqué | Dort (context switch) | Boucle active (busy-wait) |
| Overhead | Context switch coûteux | Brûle du CPU |
| Idéal si | Section critique longue | Section critique très courte |
| Utilisable dans | Userspace | Userspace (kernel : préféré) |

> [!tip] Règle de choix
> Spinlock si la section critique dure moins de quelques centaines de nanosecondes et si les threads tournent sur des CPU différents. Sinon mutex.

> [!warning] Ne jamais dormir en tenant un spinlock
> `malloc`, `printf`, tout appel système peuvent bloquer. Tenir un spinlock pendant ce temps → tous les autres threads tournent en boucle inutilement.

## pthread_spinlock_t

```c
#include <pthread.h>

pthread_spinlock_t spin;

pthread_spin_init(&spin, PTHREAD_PROCESS_PRIVATE);   // privé au processus
// ou PTHREAD_PROCESS_SHARED pour partage inter-processus (shm)

pthread_spin_lock(&spin);
// --- section critique courte ---
counter++;
// ---
pthread_spin_unlock(&spin);

int ret = pthread_spin_trylock(&spin);   // EBUSY si déjà pris

pthread_spin_destroy(&spin);
```

## pthread_barrier_t — point de rendez-vous

Une barrière bloque tous les threads jusqu'à ce qu'un nombre défini d'entre eux l'ait atteinte. Utile pour synchroniser des phases de calcul parallèle.

```
Thread 1 ──────────┐
Thread 2 ──────────┤ barrier_wait → tous repartent ensemble
Thread 3 ──────────┘
```

```c
pthread_barrier_t barrier;

pthread_barrier_init(&barrier, NULL, 3);  // 3 threads doivent attendre

void *worker(void *arg) {
    // Phase 1 — calcul indépendant
    do_work_phase1();

    pthread_barrier_wait(&barrier);   // attendre les 2 autres threads

    // Phase 2 — tous les threads ont fini la phase 1
    do_work_phase2();
    return NULL;
}

// Dans main, après pthread_join de tous les threads :
pthread_barrier_destroy(&barrier);
```

## Valeur de retour de pthread_barrier_wait

```c
int ret = pthread_barrier_wait(&barrier);
if (ret == PTHREAD_BARRIER_SERIAL_THREAD) {
    // Ce thread est le "dernier" à franchir la barrière
    // Exactement 1 thread reçoit cette valeur — utile pour du cleanup
}
// Les autres threads reçoivent 0
```

> [!tip] Usage typique
> Traitement parallèle en phases : tous les threads terminent la phase N avant que quiconque commence la phase N+1.
