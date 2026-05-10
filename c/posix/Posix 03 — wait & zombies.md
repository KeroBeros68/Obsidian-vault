#c #posix #processus #wait #zombie

## Cycle de vie d'un processus enfant

```
fork() → enfant tourne → enfant se termine → zombie
                                                  ↓
                              parent fait wait() → ressources libérées
```

Un **zombie** est un processus terminé dont les ressources ne sont pas encore récupérées par le parent. Il occupe une entrée dans la table des processus jusqu'au `wait`.

## wait & waitpid

```c
#include <sys/wait.h>

pid_t wait(int *status);
// Attend n'importe quel enfant. Bloque jusqu'à ce qu'un enfant se termine.

pid_t waitpid(pid_t pid, int *status, int options);
// pid > 0  : attendre cet enfant spécifique
// pid = -1 : attendre n'importe quel enfant (comme wait)
// pid =  0 : attendre un enfant du même groupe de processus
```

## Analyser le status de sortie

```c
int status;
pid_t pid = waitpid(child_pid, &status, 0);

if (WIFEXITED(status)) {
    int code = WEXITSTATUS(status);   // code de exit() ou return de main
    printf("Sorti normalement, code = %d\n", code);
}

if (WIFSIGNALED(status)) {
    int sig = WTERMSIG(status);       // signal qui a tué le processus
    printf("Tué par signal %d\n", sig);
}

if (WIFSTOPPED(status)) {
    int sig = WSTOPSIG(status);       // signal ayant stoppé (SIGSTOP, SIGTSTP)
    printf("Stoppé par signal %d\n", sig);
}
```

## Options de waitpid

```c
waitpid(pid, &status, WNOHANG);     // non-bloquant : retourne 0 si l'enfant n'est pas terminé
waitpid(pid, &status, WUNTRACED);   // retourne aussi pour les processus stoppés
waitpid(pid, &status, WCONTINUED);  // retourne si l'enfant a reçu SIGCONT
```

## Processus orphelin

Si le parent se termine avant l'enfant, l'enfant est **adopté par init** (PID 1) qui fait les `wait` nécessaires. Pas de zombie dans ce cas.

## Éviter les zombies avec SIGCHLD

```c
#include <signal.h>

// Ignorer SIGCHLD : le noyau nettoie automatiquement les enfants terminés
signal(SIGCHLD, SIG_IGN);

// Ou handler explicite pour récupérer les statuts
void sigchld_handler(int sig) {
    int status;
    while (waitpid(-1, &status, WNOHANG) > 0)
        ;   // vider tous les enfants terminés
}

struct sigaction sa = { .sa_handler = sigchld_handler, .sa_flags = SA_RESTART };
sigaction(SIGCHLD, &sa, NULL);
```

> [!tip] `waitpid(-1, &status, WNOHANG)` dans le handler SIGCHLD
> Toujours utiliser une boucle avec `WNOHANG` : plusieurs enfants peuvent se terminer entre deux livraisons du signal (les signaux ne se queue pas).

> [!warning] Zombie vs orphelin
> **Zombie** : enfant terminé, parent vivant mais n'a pas fait `wait` → entrée dans la table de processus non libérée.
> **Orphelin** : parent terminé avant l'enfant → adopté par init, pas de zombie.
