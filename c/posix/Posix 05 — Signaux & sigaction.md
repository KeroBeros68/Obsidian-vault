#c #posix #signaux #sigaction #async

## Signaux courants

| Signal | Valeur | Cause par défaut | Action défaut |
|--------|--------|-----------------|---------------|
| `SIGTERM` | 15 | Demande d'arrêt logiciel | Terminer |
| `SIGKILL` | 9 | Arrêt forcé (non interceptable) | Terminer |
| `SIGINT` | 2 | Ctrl+C | Terminer |
| `SIGSEGV` | 11 | Accès mémoire invalide | Core dump |
| `SIGABRT` | 6 | `abort()` | Core dump |
| `SIGCHLD` | 17 | Enfant terminé ou stoppé | Ignorer |
| `SIGPIPE` | 13 | Écriture dans pipe sans lecteur | Terminer |
| `SIGUSR1` | 10 | Usage défini par l'application | Terminer |
| `SIGUSR2` | 12 | Usage défini par l'application | Terminer |
| `SIGALRM` | 14 | Timer `alarm()` expiré | Terminer |
| `SIGSTOP` | 19 | Pause (non interceptable) | Stopper |
| `SIGCONT` | 18 | Reprendre un processus stoppé | Continuer |

## sigaction — handler moderne

```c
#include <signal.h>

struct sigaction {
    void     (*sa_handler)(int);               // handler simple
    void     (*sa_sigaction)(int, siginfo_t *, void *); // handler étendu
    sigset_t   sa_mask;    // signaux bloqués pendant l'exécution du handler
    int        sa_flags;   // options
};

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```

```c
void handler(int sig) {
    // ⚠️ uniquement des fonctions async-signal-safe ici
    write(STDERR_FILENO, "signal reçu\n", 12);  // ✅
    // printf("...");  ❌ pas async-signal-safe
}

struct sigaction sa = {
    .sa_handler = handler,
    .sa_flags   = SA_RESTART,  // relancer les appels système interrompus
};
sigemptyset(&sa.sa_mask);
sigaction(SIGTERM, &sa, NULL);
sigaction(SIGINT,  &sa, NULL);
```

## Valeurs spéciales de sa_handler

```c
signal(SIGINT, SIG_DFL);   // restaurer l'action par défaut
signal(SIGPIPE, SIG_IGN);  // ignorer le signal
```

## Masques de signaux — sigprocmask

```c
sigset_t set;
sigemptyset(&set);           // vider le masque
sigfillset(&set);            // tous les signaux
sigaddset(&set, SIGINT);     // ajouter SIGINT
sigdelset(&set, SIGINT);     // retirer SIGINT

// Bloquer temporairement des signaux
sigset_t old;
sigprocmask(SIG_BLOCK, &set, &old);    // bloquer set
// ... section critique vis-à-vis des signaux ...
sigprocmask(SIG_SETMASK, &old, NULL);  // restaurer
```

## sig_atomic_t — variable partagée avec handler

```c
volatile sig_atomic_t running = 1;   // seul type garanti safe dans un handler

void handler(int sig) {
    running = 0;
}

int main(void) {
    // ... sigaction setup ...
    while (running)
        do_work();
}
```

## Signaux et threads (lien pthread)

```c
// pthread_sigmask : comme sigprocmask mais par thread
pthread_sigmask(SIG_BLOCK, &set, &old);

// kill envoie à tout le processus (n'importe quel thread peut le recevoir)
kill(pid, SIGTERM);

// pthread_kill envoie à un thread spécifique
pthread_kill(tid, SIGUSR1);
```

> [!tip] Pattern recommandé pour les signaux en multithreadé
> Bloquer tous les signaux dans les threads workers avec `pthread_sigmask`. Dédier un thread à `sigwait()` pour traiter les signaux de façon synchrone — évite les contraintes des fonctions async-signal-safe.

> [!warning] Fonctions async-signal-safe
> Dans un handler, uniquement des fonctions de la liste POSIX async-signal-safe : `write`, `_exit`, `kill`, `sigprocmask`... `malloc`, `printf`, `pthread_mutex_lock` sont **interdits** — risque de deadlock ou corruption.
