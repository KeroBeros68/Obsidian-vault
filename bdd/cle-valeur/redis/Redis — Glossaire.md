#bdd #redis #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Valkey** | Fork de Redis (depuis la version 7.2.4) créé en 2024 suite au changement de licence, maintenu sous licence BSD par la Linux Foundation. |
| **RSALv2 / SSPLv1** | Licences *source-available* (non reconnues open source par l'OSI) proposées par Redis depuis 2024, restreignant la commercialisation en service managé. |
| **AGPLv3** | Licence open source approuvée par l'OSI, ajoutée comme troisième choix par Redis depuis la version 8.0, avec une clause réseau imposant le partage du code des modifications exposées en service. |
| **Redis Open Source** | Nom de l'édition communautaire de Redis depuis la version 8.0, incluant nativement les anciens modules Stack (JSON, recherche, séries temporelles, probabiliste). |
| **Event loop** | Boucle d'événements mono-thread traitant séquentiellement toutes les commandes Redis, via multiplexage d'E/S (`epoll`/`kqueue`). |
| **Threads d'E/S** | Threads (depuis Redis 6.0) déchargeant la lecture/écriture réseau du thread principal, sans jamais paralléliser l'exécution des commandes elles-mêmes. |
| **String** | Type de donnée fondamental de Redis : une séquence d'octets (texte, binaire, objet sérialisé), jusqu'à 512 Mo. |
| **Sorted Set** | Ensemble de valeurs uniques triées par score, structure de référence pour les classements et files de priorité. |
| **Stream** | Journal d'événements append-only avec support natif de groupes de consommateurs et livraison *at-least-once*. |
| **HyperLogLog** | Structure probabiliste estimant la cardinalité d'un ensemble avec une erreur d'environ 0,81 %, en quelques kilo-octets seulement. |
| **TTL (Time To Live)** | Durée de vie restante d'une clé avant suppression automatique — consultable via `TTL`, définie via `EXPIRE`/`SETEX`. |
| **Expiration active / passive** | Passive : vérifiée à chaque accès à une clé. Active : cycle périodique en arrière-plan qui purge les clés expirées jamais relues. |
| **`maxmemory-policy`** | Directive définissant la politique d'éviction appliquée quand `maxmemory` est atteinte (`allkeys-lru`, `volatile-ttl`, `noeviction`...). |
| **LRU approximé** | Algorithme d'éviction de Redis échantillonnant un sous-ensemble de clés au hasard, plutôt qu'un LRU exact, pour limiter le coût mémoire/CPU. |
| **LRM (Least Recently Modified)** | Politique d'éviction (Redis 8.6+) basée sur la dernière écriture d'une clé, ignorant les accès en lecture. |
| **Compteur de Morris** | Compteur probabiliste sur quelques bits, utilisé pour approximer la fréquence d'accès en mode d'éviction LFU. |
| **RDB (Redis Database)** | Fichier binaire d'instantané complet du jeu de données, produit via `SAVE`/`BGSAVE`, généré par `fork()` et copy-on-write. |
| **AOF (Append Only File)** | Journal de toutes les commandes d'écriture, rejouable au redémarrage — offre une durabilité plus fine que RDB. |
| **`BGREWRITEAOF`** | Commande recompactant l'AOF en la séquence minimale de commandes nécessaires pour reconstruire l'état courant. |
| **`MULTI`/`EXEC`** | Bloc de commandes mises en file d'attente puis exécutées atomiquement (sans entrelacement d'autres clients), sans rollback automatique en cas d'erreur applicative. |
| **`WATCH`** | Verrouillage optimiste : annule une transaction `MULTI`/`EXEC` si une clé surveillée a été modifiée entre-temps par un autre client. |
| **Redis Functions** | Bibliothèques de scripts Lua nommées et persistées (depuis Redis 7.0), alternative structurée à `EVAL`. |
| **Keyspace notifications** | Événements Pub/Sub publiés automatiquement par Redis à chaque opération sur une clé (écriture, expiration, éviction). |
| **PSYNC** | Protocole de resynchronisation d'un follower auprès de son leader, permettant une reprise partielle du flux de réplication plutôt qu'un transfert complet. |
| **ID de réplication / offset** | Identifiant de l'historique d'un jeu de données côté leader, combiné à un offset croissant qui permet de déterminer précisément l'état de synchronisation d'un follower. |
| **Réplication diskless** | Transfert direct d'un snapshot vers un follower via le réseau, sans écriture intermédiaire sur disque côté leader. |
| **Quorum (Sentinel)** | Nombre de Sentinels devant s'accorder pour marquer un leader `ODOWN` — ne suffit pas à lui seul à déclencher le failover, qui exige la majorité absolue des Sentinels. |
| **`SDOWN` / `ODOWN`** | États de panne d'un leader vus par Sentinel : subjectif (un seul Sentinel) puis objectif (quorum de Sentinels d'accord). |
| **Hash slot** | Une des 16384 divisions de l'espace des clés dans Redis Cluster, assignée à un nœud maître via `CRC16(clé) mod 16384`. |
| **Hash tag** | Portion d'une clé entre accolades (`{...}`) forçant plusieurs clés sur le même hash slot, indispensable aux opérations multi-clés en Cluster. |
| **`MOVED` / `ASK`** | Réponses de redirection d'un nœud Cluster : `MOVED` pour une clé définitivement sur un autre nœud, `ASK` pour une redirection temporaire pendant un resharding. |
| **ACL (Access Control List)** | Système de comptes nommés (depuis Redis 6.0) avec permissions par commande, catégorie et motif de clé, remplaçant l'authentification par simple mot de passe. |
| **`dir`** | Directive `redis.conf` fixant le répertoire de travail où sont écrits `dump.rdb` et le répertoire AOF — doit toujours être un chemin absolu en production. |
| **`include`** | Directive `redis.conf` permettant de fragmenter la configuration en plusieurs fichiers, inclus par ordre alphabétique en cas de motif avec joker. |
| **`CONFIG REWRITE`** | Commande qui réécrit `redis.conf` pour refléter l'état courant du serveur, de façon conservatrice (préserve commentaires et structure, n'ajoute que les valeurs non-défaut). |
| **`redis-full.conf`** | Fichier de configuration (Redis 8+) chargeant le serveur cœur (`include redis.conf`) plus les quatre composants intégrés (recherche, JSON, séries temporelles, probabiliste). |
