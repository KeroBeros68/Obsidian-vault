#c #posix #ipc #mmap #mémoire-partagée

## mmap — mapper de la mémoire

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
// addr   : NULL = noyau choisit l'adresse
// length : taille du mapping
// prot   : protection mémoire
// flags  : type de mapping
// fd, offset : fichier à mapper (MAP_ANONYMOUS → fd=-1, offset=0)

int munmap(void *addr, size_t length);
```

## Protections mémoire

```c
PROT_READ    // lecture
PROT_WRITE   // écriture
PROT_EXEC    // exécution
PROT_NONE    // aucun accès
```

## Types de mapping

```c
MAP_SHARED     // modifications visibles par les autres processus/threads (et dans le fichier)
MAP_PRIVATE    // copy-on-write : modifications locales, pas visibles ailleurs
MAP_ANONYMOUS  // pas de fichier sous-jacent (mémoire pure)
MAP_FIXED      // forcer l'adresse addr (dangereux)
```

## Mémoire anonyme partagée entre threads

```c
// Allouer de la mémoire partagée entre threads du même processus
// (équivalent à malloc mais via mmap)
int *buf = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                MAP_SHARED | MAP_ANONYMOUS, -1, 0);
if (buf == MAP_FAILED) { perror("mmap"); return 1; }

buf[0] = 42;
munmap(buf, 4096);
```

## mmap d'un fichier

```c
int fd = open("data.bin", O_RDWR);
struct stat st;
fstat(fd, &st);

char *map = mmap(NULL, st.st_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
close(fd);   // fd peut être fermé après mmap, le mapping reste valide

map[0] = 'X';    // modifie directement le fichier (MAP_SHARED)
msync(map, st.st_size, MS_SYNC);   // forcer l'écriture sur disque

munmap(map, st.st_size);
```

## Mémoire partagée POSIX entre processus — shm_open

```c
#include <sys/mman.h>
#include <fcntl.h>

// Créer ou ouvrir un objet de mémoire partagée
int fd = shm_open("/mon_shm", O_CREAT | O_RDWR, 0666);
ftruncate(fd, sizeof(int) * 100);   // définir la taille

int *shared = mmap(NULL, sizeof(int) * 100,
                   PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
close(fd);

// Utiliser shared entre processus...
shared[0] = 42;

munmap(shared, sizeof(int) * 100);
shm_unlink("/mon_shm");   // supprimer l'objet (sinon persiste jusqu'au reboot)
```

```bash
# Compiler avec
gcc prog.c -o prog -lrt
```

## Comparaison des IPC

| Mécanisme | Portée | Persistance | Usage |
|-----------|--------|-------------|-------|
| pipe | parent/enfant | processus | flux de données |
| FIFO | tous processus | filesystem | flux de données |
| `shm_open` + `mmap` | tous processus | `/dev/shm` | données structurées |
| `mmap` anonyme | threads | processus | partage rapide intra-process |

> [!warning] Synchronisation obligatoire
> La mémoire partagée ne fournit aucune synchronisation. Toujours associer un mutex, un sémaphore, ou un spinlock pour protéger les accès concurrents.

> [!tip] msync vs fsync
> `msync` synchronise le mapping vers le fichier. `fsync` synchronise le cache du noyau vers le disque. Pour garantir la durabilité : les deux.
