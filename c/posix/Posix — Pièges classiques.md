#c #posix #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier wait → zombies

```c
// ❌ enfant devient zombie si parent ne wait pas
fork();
// parent continue sans wait

// ✅ toujours récupérer les enfants
waitpid(pid, NULL, 0);

// ✅ ou ignorer SIGCHLD pour nettoyage automatique
signal(SIGCHLD, SIG_IGN);
```

> [!warning] Accumulation de zombies
> Chaque zombie occupe une entrée dans la table des processus. Si la limite est atteinte, `fork` échoue avec `EAGAIN`.

---

## 🪤 Piège 2 — fork dans un programme multithreadé

```c
// ❌ fork dans un programme avec des threads actifs
// → seul le thread appelant survit dans l'enfant
// → les mutex détenus par d'autres threads restent verrouillés
// → deadlock garanti si l'enfant appelle malloc, printf, etc.

// ✅ fork dans un programme multithreadé : appeler exec immédiatement
if (fork() == 0) {
    execvp(cmd, argv);  // exec sans délai
    _exit(1);
}
```

> [!tip] pthread_atfork
> `pthread_atfork(prepare, parent, child)` permet d'enregistrer des handlers pour acquérir/relâcher des locks autour du fork. Solution correcte mais complexe.

---

## 🪤 Piège 3 — Ne pas fermer les extrémités inutilisées d'un pipe

```c
// ❌ parent garde fd[0] ouvert → enfant ne voit jamais EOF
int fd[2];
pipe(fd);
if (fork() == 0) {
    read(fd[0], buf, sizeof(buf));  // bloque indéfiniment
}
write(fd[1], "data", 4);

// ✅ toujours fermer l'extrémité non utilisée
if (fork() == 0) {
    close(fd[1]);                  // fermer écriture côté enfant
    read(fd[0], buf, sizeof(buf));
    close(fd[0]);
}
close(fd[0]);                      // fermer lecture côté parent
write(fd[1], "data", 4);
close(fd[1]);
```

---

## 🪤 Piège 4 — exit() dans l'enfant au lieu de _exit()

```c
if (fork() == 0) {
    // ... exec échoue ...
    exit(1);    // ❌ flush les buffers stdio partagés avec le parent → double sortie

    _exit(1);   // ✅ termine sans flush
}
```

---

## 🪤 Piège 5 — SIGPIPE non géré

```c
// Écrire dans un pipe dont tous les lecteurs ont fermé leur extrémité
write(fd[1], buf, n);   // ❌ envoie SIGPIPE → termine le processus par défaut

// ✅ ignorer SIGPIPE et tester errno
signal(SIGPIPE, SIG_IGN);
if (write(fd[1], buf, n) == -1 && errno == EPIPE) {
    // gérer la déconnexion
}
```

---

## 🪤 Piège 6 — Lecture partielle non gérée

```c
// ❌ read peut retourner moins que count
read(fd, buf, 1024);   // peut retourner 50 octets même s'il y en a 1024

// ✅ boucle de lecture complète
size_t total = 0;
while (total < expected) {
    ssize_t n = read(fd, buf + total, expected - total);
    if (n <= 0) break;   // EOF ou erreur
    total += n;
}
```

---

## 🪤 Piège 7 — shm_open sans shm_unlink → fuite persistante

```c
// L'objet shm persiste dans /dev/shm jusqu'au reboot ou shm_unlink
sem_t *s = sem_open("/mon_sem", O_CREAT, 0666, 1);
// ... oubli de sem_unlink ...

// ✅ toujours unlink à la fin, même en cas d'erreur
sem_close(s);
sem_unlink("/mon_sem");

// Vérifier les fuites :
// ls /dev/shm
```

---

## 🪤 Piège 8 — Fonctions non async-signal-safe dans un handler

```c
void handler(int sig) {
    printf("signal!\n");        // ❌ printf utilise malloc en interne → deadlock possible
    pthread_mutex_lock(&mtx);   // ❌ interdit dans un handler
    free(ptr);                  // ❌ non async-signal-safe
}

void handler(int sig) {
    write(STDERR_FILENO, "signal!\n", 8);  // ✅ write est async-signal-safe
    running = 0;                            // ✅ si running est volatile sig_atomic_t
}
```

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Pas de wait → zombies | `waitpid` ou `signal(SIGCHLD, SIG_IGN)` |
| fork + threads | `exec` immédiat dans l'enfant |
| Extrémités pipe non fermées | Fermer le côté inutilisé après fork |
| `exit` dans enfant | `_exit` après un exec raté |
| SIGPIPE non géré | `SIG_IGN` + tester `EPIPE` |
| Lecture partielle | Boucle jusqu'à n octets |
| shm/sem non unlink | `shm_unlink` / `sem_unlink` à la fin |
| malloc dans handler | Seulement des fonctions async-signal-safe |
