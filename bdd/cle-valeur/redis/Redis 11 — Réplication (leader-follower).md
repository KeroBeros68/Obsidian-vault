#bdd #redis #réplication #avancé

## Un modèle leader-follower simple à configurer

```bash
# Sur le follower (redis.conf, ou à chaud)
replicaof 192.168.1.10 6379
```

```bash
INFO replication
ROLE
```

Le follower devient une copie exacte du leader et se reconnecte automatiquement si le lien réseau se coupe, en tentant de reprendre là où il s'était arrêté.

## Réplication asynchrone par défaut

Le leader ne bloque jamais l'écriture d'un client en attendant qu'un follower confirme réception — la réplication est asynchrone, privilégiant la latence et le débit. Chaque follower accuse réception périodiquement (une fois par seconde) de la quantité de données traitées.

```bash
WAIT 2 1000   # Attend l'ACK d'au moins 2 followers, ou 1000ms max
```

> [!warning] WAIT réduit le risque de perte, sans l'éliminer
> `WAIT` ne transforme pas Redis en système à cohérence forte : une écriture acquittée par `WAIT` peut malgré tout être perdue lors d'un failover, selon la configuration de persistance en place. Il réduit la fenêtre de perte de données à des cas nettement plus rares, sans l'annuler.

## Resynchronisation partielle (PSYNC) vs complète

Chaque leader possède un **ID de réplication** et un **offset** qui progresse à chaque octet de flux de réplication produit. Quand un follower se reconnecte, il annonce son ancien ID et son offset via `PSYNC` :

- Si le leader dispose encore du backlog nécessaire → **resynchronisation partielle** : seul le flux manqué est retransmis.
- Sinon → **resynchronisation complète** : le leader génère un nouveau snapshot RDB (voir [[Redis 07 — Persistance RDB (snapshots)]]), le transfère intégralement, puis reprend le flux de commandes.

```bash
# redis.conf — taille du backlog conservé pour permettre une resync partielle
repl-backlog-size 10mb
```

> [!info] La réplication sans disque (diskless replication)
> Par défaut, une resynchronisation complète passe par l'écriture d'un fichier RDB sur disque avant transfert. `repl-diskless-sync yes` fait directement transiter le snapshot par le réseau vers le follower, sans écriture disque intermédiaire côté leader — utile sur un disque lent qui pénaliserait sinon la synchronisation.

## Follower en lecture seule par défaut

```bash
CONFIG GET replica-read-only   # yes par défaut
```

Un follower refuse toute commande d'écriture par défaut, empêchant une divergence accidentelle avec le leader. Le rendre inscriptible (`replica-read-only no`) est déconseillé en dehors de cas très spécifiques : toute écriture locale est perdue à la prochaine resynchronisation et peut créer des incohérences si les mêmes clés sont aussi écrites côté leader.

## Expiration des clés sur un follower

Un follower n'expire jamais une clé de sa propre initiative : il attend que le leader envoie explicitement un `DEL` (voir [[Redis 05 — Expiration des clés (TTL)]]), évitant toute divergence liée à une désynchronisation d'horloge entre les deux instances.

## Pour aller plus loin

La réplication seule ne gère pas automatiquement le failover en cas de panne du leader — c'est le rôle de Redis Sentinel, détaillé dans [[Redis 12 — Sentinel (haute disponibilité)]].

Sources : [Redis replication — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)
