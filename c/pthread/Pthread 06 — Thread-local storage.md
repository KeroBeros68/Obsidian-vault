#c #pthread #tls #thread-local

## Principe

Le TLS (Thread-Local Storage) donne à chaque thread sa propre instance d'une variable. Les threads ne partagent pas cette donnée → pas besoin de mutex.

## `__thread` — TLS statique (GCC/Clang)

```c
__thread int counter = 0;   // chaque thread a son propre counter

void *worker(void *arg) {
    counter++;               // ne touche que la copie de ce thread
    printf("counter = %d\n", counter);   // toujours 1
    return NULL;
}
```

```c
// Cas courants
__thread int   errno_local;      // errno est déjà TLS dans glibc
__thread char  buf[256];         // buffer par thread, pas de mutex
static __thread int cache;       // TLS avec linkage interne
```

> [!info] C11 : `_Thread_local`
> Le mot-clé standard C11 est `_Thread_local`. `__thread` est l'extension GCC équivalente, plus courante en pratique.

## `pthread_key_t` — TLS dynamique POSIX

Utile quand le nombre de clés n'est pas connu à la compilation, ou pour associer un destructeur.

```c
pthread_key_t key;

// Créer la clé (une seule fois, ex: dans main ou pthread_once)
pthread_key_create(&key, free);   // free appelé sur la valeur à la fin du thread

// Dans chaque thread
void *buf = malloc(256);
pthread_setspecific(key, buf);    // associer une valeur à la clé

// Lire la valeur (NULL si jamais setspecific appelé)
char *p = pthread_getspecific(key);

// Supprimer la clé (ne détruit pas les valeurs existantes)
pthread_key_delete(key);
```

## `pthread_once` — initialisation unique

```c
pthread_once_t once = PTHREAD_ONCE_INIT;

void init_key(void) {
    pthread_key_create(&key, free);   // exécuté une seule fois
}

void *worker(void *arg) {
    pthread_once(&once, init_key);    // thread-safe, quel que soit le thread
    pthread_setspecific(key, malloc(256));
    return NULL;
}
```

## Comparaison des deux approches

| | `__thread` | `pthread_key_t` |
|--|------------|-----------------|
| Syntaxe | Simple | Verbeuse |
| Portabilité | GCC/Clang | POSIX standard |
| Destructeur | Non | Oui (via `key_create`) |
| Nombre de clés | Statique | Dynamique |
| Cas d'usage | Variables simples | Données complexes, bibliothèques |

## Cas d'usage typiques

```c
__thread int  errno;             // déjà fait par glibc
__thread char strerror_buf[256]; // buffer de formatage par thread
__thread struct db_conn *conn;   // connexion BD par thread (pas de pool mutex)
```

> [!tip] Avantage principal
> Éliminer un mutex sur des données per-thread. Si chaque thread a sa propre copie, il n'y a pas de partage à protéger.
