#c #posix #ipc #pipe #fifo

## pipe — communication unidirectionnelle

```c
#include <unistd.h>

int pipe(int pipefd[2]);
// pipefd[0] → extrémité de lecture
// pipefd[1] → extrémité d'écriture
```

```
Parent                     Enfant
write(pipefd[1]) ────────► read(pipefd[0])
```

## Exemple — parent envoie à l'enfant

```c
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(void) {
    int fd[2];
    pipe(fd);

    pid_t pid = fork();

    if (pid == 0) {
        // Enfant — lecteur
        close(fd[1]);              // fermer l'extrémité écriture
        char buf[64];
        ssize_t n = read(fd[0], buf, sizeof(buf));
        buf[n] = '\0';
        printf("Enfant reçoit : %s\n", buf);
        close(fd[0]);
        _exit(0);
    }

    // Parent — écrivain
    close(fd[0]);                  // fermer l'extrémité lecture
    const char *msg = "hello";
    write(fd[1], msg, strlen(msg));
    close(fd[1]);                  // fermeture → EOF côté enfant

    waitpid(pid, NULL, 0);
    return 0;
}
```

> [!warning] Toujours fermer les extrémités inutilisées
> Si le parent garde `fd[0]` ouvert, le `read` de l'enfant ne verra jamais EOF (il reste un lecteur potentiel). Le processus attend indéfiniment.

## Rediriger stdin/stdout avec dup2

```c
// Connecter stdout du parent à stdin de l'enfant (shell pipeline)
int fd[2];
pipe(fd);

if (fork() == 0) {
    close(fd[1]);
    dup2(fd[0], STDIN_FILENO);   // stdin ← pipe
    close(fd[0]);
    execlp("wc", "wc", "-l", NULL);
}

close(fd[0]);
dup2(fd[1], STDOUT_FILENO);      // stdout → pipe
close(fd[1]);
execlp("ls", "ls", NULL);
// équivalent à : ls | wc -l
```

## SIGPIPE — écriture dans un pipe sans lecteur

```c
// Si tous les lecteurs ont fermé fd[0], write(fd[1]) envoie SIGPIPE
// Action par défaut : terminer le processus

signal(SIGPIPE, SIG_IGN);        // ignorer pour traiter l'erreur manuellement
// write() retournera alors -1 avec errno = EPIPE
```

## FIFO — named pipe

Un FIFO est un pipe avec un nom dans le système de fichiers. Permet la communication entre processus non apparentés.

```c
#include <sys/stat.h>

mkfifo("/tmp/mon_fifo", 0666);   // créer le FIFO

// Processus A — écrivain
int fd = open("/tmp/mon_fifo", O_WRONLY);  // bloque jusqu'à ce qu'un lecteur ouvre
write(fd, "data", 4);
close(fd);

// Processus B — lecteur
int fd = open("/tmp/mon_fifo", O_RDONLY);  // bloque jusqu'à ce qu'un écrivain ouvre
char buf[64];
read(fd, buf, sizeof(buf));
close(fd);

unlink("/tmp/mon_fifo");         // supprimer le FIFO du filesystem
```

> [!info] Capacité d'un pipe
> La capacité d'un pipe est `PIPE_BUF` octets (4096 ou 65536 selon le système). Les écritures ≤ `PIPE_BUF` sont atomiques. Au-delà, les données peuvent s'entrelacer entre processus.

| | pipe | FIFO |
|--|------|------|
| Identifiant | fd uniquement | chemin filesystem |
| Processus | parent/enfant | non apparentés |
| Persistance | durée de vie du processus | jusqu'à `unlink` |
| Direction | unidirectionnel | unidirectionnel |
