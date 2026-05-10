#c #pthread #attributs #configuration

## pthread_attr_t — init et destruction

Les attributs permettent de configurer un thread avant sa création. Toujours détruire l'objet après `pthread_create`.

```c
pthread_attr_t attr;

pthread_attr_init(&attr);
// ... configurer attr ...
pthread_create(&tid, &attr, worker, NULL);
pthread_attr_destroy(&attr);    // libérer, même si create a échoué
```

## État de détachement

```c
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);  // défaut
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
```

Préférer cette approche à `pthread_detach` post-create : évite la fenêtre de temps entre create et detach.

## Taille de la stack

```c
size_t stacksize;
pthread_attr_getstacksize(&attr, &stacksize);   // lire la valeur courante
pthread_attr_setstacksize(&attr, 4 * 1024 * 1024);  // 4 Mo

// La taille minimale est PTHREAD_STACK_MIN (défini dans <limits.h>)
```

> [!info] Taille par défaut
> En général 8 Mo sur Linux. Réduire pour des centaines de threads, augmenter pour des récursions profondes.

## Politique d'ordonnancement

```c
pthread_attr_setschedpolicy(&attr, SCHED_OTHER);  // défaut — temps partagé
pthread_attr_setschedpolicy(&attr, SCHED_FIFO);   // temps réel, FIFO
pthread_attr_setschedpolicy(&attr, SCHED_RR);     // temps réel, round-robin
```

```c
// Activer les paramètres d'ordo explicites (obligatoire pour FIFO/RR)
pthread_attr_setinheritsched(&attr, PTHREAD_EXPLICIT_SCHED);
```

## Priorité

```c
struct sched_param param;
param.sched_priority = 10;    // SCHED_FIFO/RR : 1 (bas) à 99 (haut)
pthread_attr_setschedparam(&attr, &param);
```

## Tableau récapitulatif

| Fonction | Rôle |
|----------|------|
| `pthread_attr_setdetachstate` | Joinable ou detaché |
| `pthread_attr_setstacksize` | Taille de la stack |
| `pthread_attr_setschedpolicy` | SCHED_OTHER / FIFO / RR |
| `pthread_attr_setschedparam` | Priorité temps réel |
| `pthread_attr_setinheritsched` | Hériter ou expliciter l'ordo |

> [!warning] Droits root requis
> `SCHED_FIFO` et `SCHED_RR` nécessitent `CAP_SYS_NICE` ou un `setuid root`. Sans ces droits, `pthread_create` retourne `EPERM`.
