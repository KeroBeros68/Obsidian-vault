#c #posix #ipc #sémaphores #synchronisation

## Principe

Un sémaphore est un compteur entier protégé. Deux opérations atomiques :
- **wait (P)** : décrémente. Si le compteur est à 0, bloque jusqu'à ce qu'il devienne > 0.
- **post (V)** : incrémente. Réveille un thread/processus bloqué si applicable.

Sémaphore binaire (0 ou 1) ≈ mutex. Sémaphore compteur ≥ 1 → contrôle de ressources multiples.

## Deux types de sémaphores POSIX

| | Nommé (`sem_open`) | Non-nommé (`sem_init`) |
|--|--------------------|-----------------------|
| Identifiant | Nom filesystem (`/nom`) | Variable en mémoire |
| Portée | Inter-processus | Threads (ou processus via shm) |
| Persistance | Jusqu'à `sem_unlink` | Durée de vie de la variable |

## Sémaphore non-nommé (intra-process)

```c
#include <semaphore.h>

sem_t sem;

sem_init(&sem, 0, 1);   // 0 = partagé entre threads, 1 = valeur initiale
// pshared=1 → partagé entre processus via mémoire partagée

sem_wait(&sem);         // P : bloque si valeur == 0, décrémente sinon
// --- section critique ---
sem_post(&sem);         // V : incrémente, réveille un waiter

int ret = sem_trywait(&sem);   // non-bloquant, retourne EAGAIN si 0

sem_getvalue(&sem, &val);      // lire la valeur courante (indicatif)

sem_destroy(&sem);
```

## Sémaphore nommé (inter-processus)

```c
#include <semaphore.h>

// Créer ou ouvrir
sem_t *sem = sem_open("/mon_sem", O_CREAT, 0666, 1);  // valeur initiale = 1
if (sem == SEM_FAILED) { perror("sem_open"); return 1; }

sem_wait(sem);
// ... section critique partagée entre processus ...
sem_post(sem);

sem_close(sem);           // fermer la référence locale
sem_unlink("/mon_sem");   // supprimer le sémaphore du système
```

```bash
gcc prog.c -o prog -lpthread   # sem_* font partie de libpthread
```

## sem_timedwait — attente avec timeout

```c
#include <time.h>

struct timespec ts;
clock_gettime(CLOCK_REALTIME, &ts);
ts.tv_sec += 2;   // timeout dans 2 secondes

int ret = sem_timedwait(&sem, &ts);
if (ret == -1 && errno == ETIMEDOUT) {
    // délai expiré sans acquérir le sémaphore
}
```

## Sémaphore vs Mutex pthread

| | Sémaphore | Mutex pthread |
|--|-----------|---------------|
| Ownership | Aucun — n'importe qui peut poster | Le locker doit déverrouiller |
| Valeur | Compteur ≥ 0 | Binaire |
| Inter-processus | Oui (nommé ou shm) | Avec attribut `PTHREAD_PROCESS_SHARED` |
| Usage typique | Signalisation, pool de ressources | Exclusion mutuelle |

> [!tip] Signalisation sans ownership
> Le sémaphore est idéal pour signaler depuis un handler de signal ou depuis un thread à un autre : `sem_post` est async-signal-safe, `pthread_mutex_unlock` ne l'est pas.

> [!warning] sem_init avec pshared=1
> Nécessite que la variable `sem_t` soit dans une zone de mémoire partagée (via `mmap` ou `shm_open`). Une variable locale ou globale ordinaire ne fonctionne qu'entre threads du même processus.
