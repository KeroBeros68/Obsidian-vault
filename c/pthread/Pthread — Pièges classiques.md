#c #pthread #pièges #erreurs #debugging

## 🪤 Piège 1 — Retourner l'adresse d'une variable locale

```c
void *worker(void *arg) {
    int result = 42;
    return &result;   // ❌ stack détruite à la fin du thread
}

void *worker(void *arg) {
    int *result = malloc(sizeof(int));
    *result = 42;
    return result;    // ✅ heap — à libérer par le thread qui join
}
```

> [!warning] Use-after-free
> Le thread appelant lit de la mémoire libérée. Résultat imprévisible, pas forcément un crash immédiat.

---

## 🪤 Piège 2 — `if` au lieu de `while` avant `pthread_cond_wait`

```c
if (!ready)                        // ❌ spurious wakeup possible
    pthread_cond_wait(&cond, &mtx);

while (!ready)                     // ✅ réévalue après chaque réveil
    pthread_cond_wait(&cond, &mtx);
```

> [!warning] Spurious wakeup
> POSIX autorise `pthread_cond_wait` à se réveiller sans signal. Avec `if`, le thread continue alors que `ready` est encore 0.

---

## 🪤 Piège 3 — Passer l'adresse d'une variable de boucle

```c
for (int i = 0; i < N; i++)
    pthread_create(&tid[i], NULL, worker, &i);  // ❌ tous les threads voient le même &i

// Solution : passer la valeur via un tableau ou allouer dynamiquement
int *args = malloc(N * sizeof(int));
for (int i = 0; i < N; i++) {
    args[i] = i;
    pthread_create(&tid[i], NULL, worker, &args[i]);  // ✅
}
```

---

## 🪤 Piège 4 — Double pthread_join

```c
pthread_join(tid, NULL);   // ✅
pthread_join(tid, NULL);   // ❌ undefined behavior — tid déjà libéré
```

> [!warning] UB silencieux
> Pas forcément un crash immédiat. Peut corrompre la mémoire ou bloquer indéfiniment.

---

## 🪤 Piège 5 — Mutex non relâché sur erreur

```c
pthread_mutex_lock(&mtx);
int *p = malloc(sizeof(int));
if (!p)
    return NULL;              // ❌ mtx jamais unlocked → deadlock

// Solutions :
pthread_mutex_lock(&mtx);
int *p = malloc(sizeof(int));
if (!p) {
    pthread_mutex_unlock(&mtx);   // ✅ explicite
    return NULL;
}

// Ou avec cleanup :
pthread_cleanup_push((void*)pthread_mutex_unlock, &mtx);
// ... code pouvant échouer ...
pthread_cleanup_pop(1);           // ✅ unlock garanti
```

---

## 🪤 Piège 6 — `main()` retourne sans join ni `pthread_exit`

```c
int main(void) {
    pthread_create(&tid, NULL, worker, NULL);
    return 0;   // ❌ processus tué, thread interrompu
}

int main(void) {
    pthread_create(&tid, NULL, worker, NULL);
    pthread_join(tid, NULL);   // ✅ attend la fin
    return 0;
}
```

---

## 🪤 Piège 7 — Comparer `pthread_t` avec `==`

```c
if (tid1 == tid2) { ... }              // ❌ pthread_t peut être une struct

if (pthread_equal(tid1, tid2)) { ... } // ✅
```

---

## 🪤 Piège 8 — Tenir un spinlock pendant un appel bloquant

```c
pthread_spin_lock(&spin);
printf("...");          // ❌ peut bloquer → tous les autres threads tournent en boucle
malloc(256);            // ❌ idem
pthread_spin_unlock(&spin);

// Spinlock uniquement pour des opérations garanties non-bloquantes
pthread_spin_lock(&spin);
counter++;              // ✅ arithmétique pure
pthread_spin_unlock(&spin);
```

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Retourner variable locale | Allouer sur le heap |
| `if` avant `cond_wait` | `while` obligatoire |
| `&i` dans boucle de create | Tableau d'args ou alloc dynamique |
| Double `join` | Nullifier `tid` après join |
| Mutex non relâché sur erreur | `cleanup_push` ou unlock explicite |
| `main` return sans join | `pthread_join` ou `pthread_exit` |
| Comparer `pthread_t` avec `==` | `pthread_equal` |
| Appel bloquant sous spinlock | Utiliser mutex à la place |
