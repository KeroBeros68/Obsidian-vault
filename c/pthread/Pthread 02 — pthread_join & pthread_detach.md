#c #pthread #cycle-de-vie #join #detach

## pthread_join — attendre un thread

```c
int pthread_join(pthread_t thread, void **retval);
// retval : reçoit la valeur retournée par le thread (NULL pour ignorer)
// Retourne 0 si succès, code d'erreur sinon
```

```c
void *worker(void *arg) {
    int *result = malloc(sizeof(int));
    *result = 42;
    return result;           // retourner un pointeur heap ✅
}

int main(void) {
    pthread_t tid;
    void *ret;

    pthread_create(&tid, NULL, worker, NULL);
    pthread_join(tid, &ret);

    printf("Résultat : %d\n", *(int *)ret);
    free(ret);               // libérer ce que le thread a alloué ✅
    return 0;
}
```

> [!warning] Ne jamais retourner l'adresse d'une variable locale du thread
> La stack du thread est détruite à sa fin. Le pointeur devient invalide.

## pthread_detach — thread autonome

Un thread detaché libère ses ressources seul à sa fin. `join` devient impossible.

```c
pthread_detach(tid);              // après create, depuis un autre thread
pthread_detach(pthread_self());   // ou depuis le thread lui-même
```

## Joinable vs Detached

| | Joinable (défaut) | Detached |
|--|-------------------|----------|
| Ressources à la fin | Conservées jusqu'au join | Libérées automatiquement |
| `pthread_join` | Obligatoire | Interdit |
| Valeur de retour | Récupérable | Ignorée |
| Usage | Tâche dont on attend le résultat | Tâche feu et oubli |

## main() et les threads

```c
// ❌ main retourne → processus tué, threads interrompus
int main(void) {
    pthread_create(&tid, NULL, worker, NULL);
    return 0;
}

// ✅ pthread_exit dans main : main se termine, les threads continuent
int main(void) {
    pthread_create(&tid, NULL, worker, NULL);
    pthread_exit(NULL);
}
```

## pthread_self & pthread_equal

```c
pthread_t pthread_self(void);                      // id du thread courant
int pthread_equal(pthread_t t1, pthread_t t2);     // comparaison — pas ==
```

> [!warning] Ne jamais comparer des `pthread_t` avec `==`
> Le type `pthread_t` peut être une struct selon l'implémentation. Utiliser `pthread_equal`.

> [!tip] Règle absolue
> Tout thread doit être soit **joiné**, soit **detaché**. Un thread joinable non joiné après sa fin est un thread zombie : ses ressources (stack, descripteur) restent allouées.
