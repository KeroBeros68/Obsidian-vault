#c #posix #processus #exec

## Principe

La famille `exec` **remplace** l'image du processus courant par un nouveau programme. Le PID reste identique. En cas de succès, `exec` ne retourne jamais.

```
fork()  →  enfant hérite du parent
exec()  →  enfant devient un nouveau programme
```

## La famille exec

```c
#include <unistd.h>

// Chemin absolu, args comme tableau
int execv(const char *path, char *const argv[]);

// Chemin absolu, args variadiques
int execl(const char *path, const char *arg0, ..., NULL);

// Recherche dans $PATH, args tableau
int execvp(const char *file, char *const argv[]);

// Recherche dans $PATH, args variadiques
int execlp(const char *file, const char *arg0, ..., NULL);

// Chemin absolu, args tableau, environnement explicite
int execve(const char *path, char *const argv[], char *const envp[]);
```

| Suffixe | Signification |
|---------|---------------|
| `v` | argv comme tableau (`char *[]`) |
| `l` | argv comme liste variadique |
| `p` | recherche dans `$PATH` |
| `e` | environnement explicite (`envp`) |

## Exemple — fork + exec (spawn)

```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdio.h>

int main(void) {
    pid_t pid = fork();

    if (pid == 0) {
        // Enfant : remplacer par ls -l
        char *argv[] = {"ls", "-l", "/tmp", NULL};
        execvp("ls", argv);
        perror("execvp");   // atteint seulement si exec échoue
        _exit(1);           // _exit, pas exit (ne flush pas les buffers du parent)
    }

    int status;
    waitpid(pid, &status, 0);
    printf("ls terminé avec code %d\n", WEXITSTATUS(status));
    return 0;
}
```

## Ce qui est conservé après exec

| Conservé | Perdu |
|----------|-------|
| PID, PPID | Mémoire (code, data, heap, stack) |
| Descripteurs de fichiers (sauf `O_CLOEXEC`) | Handlers de signaux personnalisés |
| UID, GID | Threads (tous sauf le courant) |
| Répertoire courant | Mappings mémoire (mmap) |

> [!tip] O_CLOEXEC — fermeture automatique au exec
> Ouvrir un fd avec `O_CLOEXEC` (ou `fcntl(fd, F_SETFD, FD_CLOEXEC)`) le ferme automatiquement à l'exec. Bonne pratique pour éviter les fuites de fd dans les processus enfants.

> [!warning] Toujours utiliser `_exit` dans l'enfant après un exec raté
> `exit()` flush les buffers stdio, qui sont partagés avec le parent après `fork`. Utiliser `_exit()` pour éviter une double écriture.
