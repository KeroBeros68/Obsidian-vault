#c #posix #temps #timer #horloge

## Les horloges POSIX

```c
#include <time.h>

struct timespec {
    time_t tv_sec;    // secondes
    long   tv_nsec;   // nanosecondes [0, 999999999]
};

int clock_gettime(clockid_t clk_id, struct timespec *tp);
```

| Horloge | Usage | Caractéristique |
|---------|-------|-----------------|
| `CLOCK_REALTIME` | Heure du monde réel | Ajustable (NTP, settimeofday) |
| `CLOCK_MONOTONIC` | Mesure de durée | Ne recule jamais, pas d'epoch fixe |
| `CLOCK_PROCESS_CPUTIME_ID` | Temps CPU du processus | Somme de tous les threads |
| `CLOCK_THREAD_CPUTIME_ID` | Temps CPU du thread courant | Par thread |

> [!tip] Mesure de performance → CLOCK_MONOTONIC
> `CLOCK_REALTIME` peut sauter (ajustement NTP, heure d'été). `CLOCK_MONOTONIC` est garanti croissant → toujours utiliser pour les mesures de durée.

## Mesurer une durée

```c
struct timespec start, end;

clock_gettime(CLOCK_MONOTONIC, &start);
do_work();
clock_gettime(CLOCK_MONOTONIC, &end);

long ns = (end.tv_sec - start.tv_sec) * 1000000000L
        + (end.tv_nsec - start.tv_nsec);
printf("Durée : %ld ns (%.3f ms)\n", ns, ns / 1e6);
```

## nanosleep — dormir précisément

```c
#include <time.h>

struct timespec req = { .tv_sec = 0, .tv_nsec = 500000000 };  // 500 ms
struct timespec rem;

int ret = nanosleep(&req, &rem);
// ret == -1 && errno == EINTR → interrompu par signal, rem contient le reste
if (ret == -1 && errno == EINTR)
    nanosleep(&rem, NULL);   // continuer si nécessaire
```

## Timeout sur pthread_cond_timedwait

```c
// Calculer l'heure limite absolue (obligatoire : CLOCK_REALTIME)
struct timespec deadline;
clock_gettime(CLOCK_REALTIME, &deadline);
deadline.tv_sec += 5;   // timeout dans 5 secondes

pthread_mutex_lock(&mtx);
while (!ready) {
    int ret = pthread_cond_timedwait(&cond, &mtx, &deadline);
    if (ret == ETIMEDOUT) {
        pthread_mutex_unlock(&mtx);
        // gérer le timeout
        return NULL;
    }
}
pthread_mutex_unlock(&mtx);
```

> [!warning] pthread_cond_timedwait utilise CLOCK_REALTIME par défaut
> Si l'horloge système est ajustée, le timeout est affecté. Pour utiliser `CLOCK_MONOTONIC`, configurer l'attribut `pthread_condattr_setclock`.

## clock_nanosleep — dormir sur une horloge précise

```c
// Dormir jusqu'à une date absolue (évite la dérive sur les boucles)
struct timespec next;
clock_gettime(CLOCK_MONOTONIC, &next);

while (running) {
    next.tv_nsec += 10000000;   // +10 ms
    if (next.tv_nsec >= 1000000000L) {
        next.tv_nsec -= 1000000000L;
        next.tv_sec++;
    }
    clock_nanosleep(CLOCK_MONOTONIC, TIMER_ABSTIME, &next, NULL);
    do_periodic_work();
}
```

## Résolution d'une horloge

```c
struct timespec res;
clock_getres(CLOCK_MONOTONIC, &res);
printf("Résolution : %ld ns\n", res.tv_nsec);   // souvent 1 ns sur Linux
```

## Récapitulatif des fonctions

| Fonction | Rôle |
|----------|------|
| `clock_gettime` | Lire l'heure d'une horloge |
| `clock_getres` | Résolution d'une horloge |
| `nanosleep` | Dormir un intervalle relatif |
| `clock_nanosleep` | Dormir jusqu'à une date absolue |
| `pthread_cond_timedwait` | Attendre une condition avec deadline |
| `sem_timedwait` | Attendre un sémaphore avec deadline |
