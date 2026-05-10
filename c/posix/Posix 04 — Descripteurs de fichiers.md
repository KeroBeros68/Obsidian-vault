#c #posix #fichiers #descripteurs #fd

## Principe

Un descripteur de fichier (fd) est un entier qui représente une ressource ouverte : fichier, pipe, socket, device. Le noyau maintient une table par processus.

```
fd 0 → stdin   (STDIN_FILENO)
fd 1 → stdout  (STDOUT_FILENO)
fd 2 → stderr  (STDERR_FILENO)
fd 3 → premier fd disponible après open()
```

## open, read, write, close

```c
#include <fcntl.h>
#include <unistd.h>

int  open(const char *path, int flags, mode_t mode);
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
int  close(int fd);
```

## Flags d'ouverture

```c
// Mode d'accès (obligatoire, un seul)
O_RDONLY    // lecture seule
O_WRONLY    // écriture seule
O_RDWR      // lecture + écriture

// Options (combinables avec |)
O_CREAT     // créer si inexistant  (mode obligatoire)
O_TRUNC     // tronquer à 0 si existant
O_APPEND    // toujours écrire en fin de fichier
O_NONBLOCK  // opérations non-bloquantes
O_CLOEXEC   // fermer automatiquement au exec
```

```c
// Créer ou tronquer pour écriture, permissions rw-r--r--
int fd = open("fichier.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
if (fd == -1) { perror("open"); return 1; }

write(fd, "hello\n", 6);
close(fd);
```

## lseek — repositionner le curseur

```c
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
// whence : SEEK_SET (début), SEEK_CUR (position courante), SEEK_END (fin)
```

```c
lseek(fd, 0, SEEK_SET);          // retour au début
lseek(fd, 0, SEEK_END);          // fin du fichier
off_t pos = lseek(fd, 0, SEEK_CUR);  // lire la position courante
```

## dup & dup2 — dupliquer un descripteur

```c
int dup(int oldfd);              // retourne le plus petit fd libre pointant vers oldfd
int dup2(int oldfd, int newfd);  // newfd pointe vers oldfd (ferme newfd si ouvert)
```

```c
// Rediriger stdout vers un fichier
int fd = open("out.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
dup2(fd, STDOUT_FILENO);   // stdout → fichier
close(fd);                 // fermer l'original (stdout pointe toujours dessus)
printf("Ceci va dans out.txt\n");
```

## Comportement de read et write

```c
// read : peut retourner moins que count (lecture partielle) ⚠️
ssize_t n = read(fd, buf, sizeof(buf));
// n == 0  → EOF
// n == -1 → erreur (vérifier errno)
// n  > 0  → octets lus, peut être < sizeof(buf)

// Lecture complète robuste
ssize_t read_all(int fd, void *buf, size_t count) {
    size_t done = 0;
    while (done < count) {
        ssize_t n = read(fd, (char*)buf + done, count - done);
        if (n <= 0) return n;
        done += n;
    }
    return done;
}
```

> [!warning] Toujours vérifier la valeur de retour de `read` et `write`
> `write` peut écrire moins d'octets que demandé (ex: pipe plein, signal reçu). Ne jamais supposer que tout a été écrit en un seul appel.

> [!tip] fd leak
> Chaque `open` sans `close` consomme une entrée dans la table. La limite par processus est `RLIMIT_NOFILE` (souvent 1024 ou 4096). Utiliser `O_CLOEXEC` pour les fd non nécessaires dans les enfants.
