#bdd #redis #fondamentaux

## Un seul thread pour l'exécution des commandes

Le cœur de Redis exécute les commandes sur un **seul thread**, via une boucle d'événements (*event loop*) basée sur le multiplexage d'E/S (`epoll` sous Linux, `kqueue` sous macOS/BSD). Ce thread unique surveille en continu toutes les connexions clientes ouvertes et traite les commandes une par une, sans jamais s'exécuter en parallèle sur plusieurs cœurs pour cette partie du travail.

> [!info] Pourquoi ce choix plutôt qu'un modèle multi-thread classique
> Un seul thread d'exécution élimine par construction les conditions de course sur les données et le besoin de verrous internes : chaque commande s'exécute jusqu'au bout avant que la suivante ne commence, garantissant l'atomicité de toute commande individuelle sans coût de synchronisation. Redis est généralement limité par la mémoire ou le réseau, pas par le CPU, tant que les commandes utilisées restent efficaces (voir [[Redis 06 — Politiques d'éviction (maxmemory-policy)]] pour un exemple de commande à éviter : les opérations en O(N) sur de grosses collections).

## Le multiplexage d'E/S : gérer des milliers de connexions sans thread par client

Plutôt que de créer un thread ou un processus par connexion cliente (coûteux en changement de contexte), Redis utilise la bibliothèque interne `ae` (*async events*), qui interroge le système d'exploitation pour savoir quelles sockets ont des données prêtes à être lues ou écrites, et ne traite que celles-là — un modèle qui passe à l'échelle jusqu'à des dizaines de milliers de connexions simultanées sur une seule instance.

```
Client A ─┐
Client B ─┼─► epoll/kqueue ─► event loop (1 thread) ─► exécution séquentielle des commandes
Client C ─┘
```

## Le mythe du « tout est single-thread »

Depuis Redis 4.0, certaines tâches lourdes (libération mémoire de grosses structures, par exemple via `UNLINK` plutôt que `DEL`) sont déportées vers des threads d'arrière-plan, pour ne pas bloquer le thread principal. Depuis Redis 6.0, un mécanisme de **threads d'E/S** (*I/O threading*) permet de paralléliser la lecture et l'écriture réseau (lecture des requêtes, écriture des réponses) sur plusieurs threads — mais l'exécution des commandes elle-même reste strictement séquentielle sur le thread principal.

> [!warning] Le threading réseau n'introduit pas de concurrence sur les données
> Les threads d'E/S de Redis 6+ ne font que lire/écrire sur les sockets en parallèle ; ils ne modifient jamais les structures de données en parallèle. La garantie d'atomicité par commande reste intacte — ce n'est qu'une optimisation pour saturer la bande passante réseau sur du matériel moderne, pas un changement de modèle de concurrence.

```bash
# Activer les threads d'E/S (redis.conf)
io-threads 4
io-threads-do-reads yes
```

## Conséquence pratique : attention aux commandes lentes

Une seule commande coûteuse (un `KEYS *` sur des millions de clés, un `SORT` sur une grosse liste) bloque tout le thread principal — donc **toutes** les autres commandes — pendant toute sa durée d'exécution.

> [!tip] Préférer les commandes qui scannent progressivement
> `SCAN` (et ses variantes `HSCAN`, `SSCAN`, `ZSCAN`) parcourt le jeu de clés par petits lots sans bloquer le serveur, contrairement à `KEYS *` qui doit tout parcourir en une seule commande atomique.

## Pour aller plus loin

Les structures de données que Redis manipule — bien au-delà d'un simple stockage clé-valeur — sont présentées dans [[Redis 03 — Types de données fondamentaux]].

Sources : [How Redis achieve concurrent operation with single thread — Medium](https://codescoddler.medium.com/how-redis-achieve-concurrent-operation-with-single-thread-e0c8d5e33bc3), [Next Generation Cloud-native In-Memory Stores: From Redis to Valkey and Beyond (arXiv)](https://arxiv.org/pdf/2510.19805)
