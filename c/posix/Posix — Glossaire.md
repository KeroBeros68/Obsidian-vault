#c #posix #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **processus** | Instance de programme en cours d'exécution avec son propre espace d'adressage, PID, et ressources. |
| **PID** | *Process ID.* Identifiant unique d'un processus dans le système. |
| **PPID** | *Parent PID.* PID du processus parent. |
| **fork** | Appel système dupliquant le processus courant. Retourne 0 dans l'enfant, PID enfant dans le parent. |
| **exec** | Famille d'appels système remplaçant l'image du processus courant par un nouveau programme. |
| **spawn** | Pattern fork + exec : créer un enfant et le remplacer par un autre programme. |
| **zombie** | Processus terminé dont le parent n'a pas encore appelé `wait`. Son entrée dans la table des processus est conservée. |
| **orphelin** | Processus dont le parent est terminé. Adopté par init (PID 1). |
| **copy-on-write (COW)** | Optimisation post-fork : parent et enfant partagent les pages mémoire jusqu'à une écriture, déclenchant alors une copie. |
| **descripteur de fichier (fd)** | Entier représentant une ressource ouverte (fichier, pipe, socket, device). Géré par la table de fd du processus. |
| **fd leak** | Descripteur de fichier ouvert non fermé. Épuise la limite `RLIMIT_NOFILE`. |
| **O_CLOEXEC** | Flag fd : fermeture automatique du fd lors d'un `exec`. Évite les fuites dans les processus enfants. |
| **signal** | Notification asynchrone envoyée à un processus ou thread. Peut être ignorée, interceptée, ou avoir son action par défaut. |
| **handler de signal** | Fonction appelée à la réception d'un signal. Contrainte aux fonctions async-signal-safe. |
| **async-signal-safe** | Fonction dont l'appel depuis un handler de signal est garanti sans deadlock ni corruption (ex: `write`, `_exit`, `kill`). |
| **sig_atomic_t** | Type entier dont la lecture/écriture est atomique vis-à-vis des signaux. Seul type garanti safe dans un handler. |
| **masque de signaux** | Ensemble de signaux bloqués pour un thread. Signaux bloqués mis en attente jusqu'au déblocage. |
| **pipe** | Canal de communication unidirectionnel entre processus. Deux fd : lecture et écriture. |
| **FIFO** | *Named pipe.* Pipe avec un nom dans le filesystem. Permet la communication entre processus non apparentés. |
| **SIGPIPE** | Signal envoyé lors d'une écriture dans un pipe sans lecteur. Action défaut : terminer le processus. |
| **mmap** | Mappage d'un fichier ou de mémoire anonyme dans l'espace d'adressage du processus. |
| **MAP_SHARED** | Modifications mmap visibles par les autres processus mappant la même région. |
| **MAP_PRIVATE** | Modifications mmap locales (copy-on-write), non visibles ailleurs. |
| **shm_open** | Crée ou ouvre un objet de mémoire partagée POSIX identifié par un nom. |
| **sémaphore** | Compteur entier protégé avec deux opérations atomiques : wait (décrémente) et post (incrémente). |
| **CLOCK_REALTIME** | Horloge du monde réel, ajustable par NTP. À éviter pour les mesures de durée. |
| **CLOCK_MONOTONIC** | Horloge garantie croissante. Référence pour mesurer des durées. |
| **IPC** | *Inter-Process Communication.* Mécanismes permettant la communication entre processus : pipes, FIFOs, shm, sémaphores, sockets. |
