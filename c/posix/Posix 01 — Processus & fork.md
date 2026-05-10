#c #posix #processus #fork #bases

## Qu'est-ce qu'un processus

Un processus est une instance de programme en cours d'exécution avec son propre espace d'adressage virtuel, ses descripteurs de fichiers, et son état.

```
Processus
├── PID    — identifiant unique
├── PPID   — PID du parent
├── UID    — propriétaire
├── Espace mémoire virtuel (code, data, heap, stack)
├── Table de descripteurs de fichiers
└── Handlers de signaux
```

```c
#include <unistd.h>

pid_t getpid(void);    // PID du processus courant
pid_t getppid(void);   // PID du parent
uid_t getuid(void);    // UID de l'utilisateur
```

## fork — dupliquer un processus

```c
#include <unistd.h>

pid_t fork(void);
// Dans le parent : retourne le PID de l'enfant (> 0)
// Dans l'enfant  : retourne 0
// Erreur         : retourne -1 (EAGAIN, ENOMEM)
```

Après `fork`, les deux processus sont identiques : même code, même mémoire (copy-on-write), même descripteurs de fichiers ouverts.

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    pid_t pid = fork();

    if (pid < 0) {
        perror("fork");
        return 1;
    } else if (pid == 0) {
        // Processus enfant
        printf("Enfant : PID=%d, parent=%d\n", getpid(), getppid());
    } else {
        // Processus parent
        printf("Parent : PID=%d, enfant=%d\n", getpid(), pid);
    }
    return 0;
}
```

## Copy-on-write (COW)

Après `fork`, parent et enfant partagent physiquement les mêmes pages mémoire. Une copie n'est faite que lors d'une écriture.

```
fork()
  ├── Parent : heap page A → [x=1]  ← lecture seule partagée
  └── Enfant : heap page A → [x=1]

Enfant écrit x=2 :
  ├── Parent : page A → [x=1]  ← inchangée
  └── Enfant : page A' → [x=2] ← nouvelle copie
```

> [!info] Héritage après fork
> L'enfant hérite des descripteurs de fichiers ouverts, des handlers de signaux, et du répertoire courant. Les locks mutex ne sont **pas** hérités de façon sûre (voir piège fork+threads).

> [!warning] fork dans un programme multithreadé
> Seul le thread appelant `fork` est reproduit dans l'enfant. Les autres threads disparaissent, mais leurs locks mutex peuvent rester verrouillés → deadlock garanti si l'enfant les utilise. Appeler uniquement des fonctions async-signal-safe après fork dans l'enfant, puis `exec` immédiatement.
