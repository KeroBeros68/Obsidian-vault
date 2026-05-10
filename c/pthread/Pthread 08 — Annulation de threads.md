#c #pthread #cancel #annulation

## pthread_cancel — demander l'arrêt d'un thread

```c
pthread_cancel(tid);   // envoie une demande d'annulation au thread
// Retourne 0 si la demande a été envoyée (pas si le thread est mort)
```

`pthread_cancel` ne tue pas immédiatement. Le thread est annulé seulement à un **point d'annulation**, selon son état.

## États d'annulation

```c
// Activer ou désactiver l'annulation
pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);   // défaut
pthread_setcancelstate(PTHREAD_CANCEL_DISABLE, NULL);  // ignore les cancel

// Type d'annulation
pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, NULL);     // défaut
pthread_setcanceltype(PTHREAD_CANCEL_ASYNCHRONOUS, NULL); // immédiat ⚠️
```

> [!warning] `PTHREAD_CANCEL_ASYNCHRONOUS` est dangereux
> Le thread peut être tué à n'importe quelle instruction, même au milieu d'un `malloc` ou d'un `free`. Éviter sauf cas très contrôlé (calcul pur sans ressources).

## Points d'annulation — DEFERRED

Le thread n'est annulé qu'en atteignant un point d'annulation. Fonctions courantes qui en sont :

```
pthread_join        pthread_cond_wait    sleep / nanosleep
pthread_testcancel  read / write         open / close
accept              recv / send          ...
```

```c
void *worker(void *arg) {
    while (1) {
        pthread_testcancel();    // point d'annulation explicite
        do_cpu_intensive_work(); // pas de point d'annulation ici
    }
    return NULL;
}
```

## Nettoyage — pthread_cleanup_push / pop

Permet d'enregistrer des fonctions appelées automatiquement si le thread est annulé ou appelle `pthread_exit`.

```c
void cleanup(void *arg) {
    pthread_mutex_t *m = arg;
    pthread_mutex_unlock(m);   // libérer le verrou même en cas d'annulation
    free(arg);
}

void *worker(void *arg) {
    pthread_mutex_lock(&mtx);
    pthread_cleanup_push(cleanup, &mtx);  // enregistrer le nettoyage

    do_work();   // si cancel ici → cleanup() est appelé

    pthread_cleanup_pop(1);   // 1 = exécuter cleanup maintenant
    // 0 = dépiler sans exécuter
    return NULL;
}
```

> [!warning] `pthread_cleanup_push/pop` sont des macros
> Elles doivent être dans le **même bloc** `{}` et appairées. `push` sans `pop` dans le même scope = erreur de compilation.

## Récupérer la valeur d'annulation

```c
void *ret;
pthread_join(tid, &ret);
if (ret == PTHREAD_CANCELED) {
    // le thread a bien été annulé
}
```

> [!tip] Alternative plus sûre
> Préférer un flag partagé `volatile int stop = 0` avec vérification explicite dans le thread plutôt que `pthread_cancel`. Plus prévisible, pas de gestion de points d'annulation.
