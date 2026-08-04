#bdd #redis #avancé

## Publier et s'abonner à un canal

```bash
# Client A
SUBSCRIBE canal_notifications

# Client B
PUBLISH canal_notifications "nouvel événement"
```

Un message publié sur un canal est transmis instantanément à tous les abonnés connectés au moment de la publication — Redis ne conserve **aucun historique** : un client qui s'abonne après une publication ne la reçoit jamais.

> [!warning] Pub/Sub n'est pas un système de file de messages persistant
> Contrairement à un Stream (voir [[Redis 04 — Types de données avancés]]) ou à une véritable file de messages (Kafka, RabbitMQ), un message Pub/Sub non reçu par un abonné connecté est perdu définitivement. Pour un besoin de relecture, d'accusé de réception ou de garantie de livraison, préférer les Streams et leurs groupes de consommateurs.

## Abonnement par motif (pattern)

```bash
PSUBSCRIBE evenements.*
-- Reçoit les messages publiés sur evenements.commande, evenements.paiement, etc.
```

## Keyspace notifications : Pub/Sub branché sur les clés elles-mêmes

Redis peut publier automatiquement un événement à chaque opération sur une clé (écriture, expiration, suppression), sans code applicatif supplémentaire :

```bash
CONFIG SET notify-keyspace-events KEA
-- K = keyspace (canal par clé), E = keyevent (canal par type d'événement), A = tous les événements

SUBSCRIBE __keyspace@0__:session:42     # Tout événement touchant cette clé précise
SUBSCRIBE __keyevent@0__:expired        # Toute clé qui expire, dans la base 0
```

| Lettre | Type d'événements activés |
|--------|------------------------------|
| `K` | Canal `__keyspace@<db>__:<clé>` |
| `E` | Canal `__keyevent@<db>__:<événement>` |
| `g` | Commandes génériques (`DEL`, `EXPIRE`, `RENAME`...) |
| `x` | Événements d'expiration |
| `e` | Événements d'éviction (sous pression mémoire) |
| `A` | Alias pour `g$lshzxet` (quasiment tout) |

> [!tip] Cas d'usage : invalider un cache applicatif en aval
> Un service qui maintient son propre cache local peut s'abonner à `__keyevent@0__:expired` et `__keyevent@0__:del` pour invalider ses entrées correspondantes dès que la donnée source change dans Redis, sans polling.

## Pour aller plus loin

Un seul serveur Redis reste un point de défaillance unique — la réplication vers d'autres instances est détaillée dans [[Redis 11 — Réplication (leader-follower)]].

Sources : [Redis data types — Redis Documentation](https://redis.io/docs/latest/develop/data-types/)
