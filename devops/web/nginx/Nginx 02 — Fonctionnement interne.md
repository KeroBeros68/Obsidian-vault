#devops #nginx #bases

## Le modèle événementiel

Un serveur classique (Apache en mode `prefork`) alloue un processus ou un thread par connexion : chaque connexion attend, bloquée, qu'une opération (lecture disque, réponse d'un backend) se termine. Nginx fonctionne différemment : chaque worker exécute une **boucle d'événements** (event loop) asynchrone et non bloquante, capable de gérer des milliers de connexions en attente sans leur dédier un thread chacune.

## Propriétés du modèle événementiel

- Un worker ne bloque jamais sur une connexion : il délègue les opérations lentes (I/O disque, réseau) au système d'exploitation (`epoll` sur Linux, `kqueue` sur BSD) et traite une autre connexion pendant l'attente.
- Le nombre de workers est indépendant du nombre de connexions — c'est pour ça qu'un seul worker peut tenir `worker_connections` connexions simultanées.
- Le master process ne traite aucune connexion : il ne fait que lire la configuration, ouvrir les sockets d'écoute, et gérer le cycle de vie des workers (démarrage, arrêt, reload).

## Illustration

```
Modèle process/thread-per-connection (Apache prefork)
[connexion 1] → [processus 1]  (bloqué en attente)
[connexion 2] → [processus 2]  (bloqué en attente)
[connexion 3] → [processus 3]  (bloqué en attente)
→ 1 processus = 1 connexion = mémoire × N

Modèle événementiel (Nginx)
[connexion 1] ┐
[connexion 2] ├──▶ [worker : boucle d'événements] ──▶ epoll/kqueue
[connexion 3] ┘        (aucun thread dédié par connexion)
```

## Cas particuliers

> [!warning] Une tâche CPU-bound bloque quand même le worker
> Le modèle événementiel évite les blocages liés aux attentes I/O, mais une opération lourde en calcul (déchiffrement TLS, compression gzip d'un gros fichier) occupe le worker pendant son exécution — les autres connexions de ce worker attendent. C'est pourquoi `worker_processes` est généralement aligné sur le nombre de cœurs CPU disponibles (`worker_processes auto`).

> [!tip] Pourquoi le C10K est résolu
> Avec ce modèle, la limite de connexions simultanées dépend de la mémoire disponible (chaque connexion event-driven coûte quelques Ko) et non du nombre de threads que l'OS peut ordonnancer — ce qui permet de tenir des dizaines de milliers de connexions avec un nombre de processus très inférieur au nombre de connexions.
