#bdd #redis #réplication #avancé

## Un processus séparé qui surveille et déclenche le failover

Redis Sentinel est un processus distinct (`redis-sentinel`), déployé en plusieurs exemplaires, qui surveille un ou plusieurs couples leader-follower et déclenche automatiquement une promotion en cas de panne du leader — la réplication de base (voir [[Redis 11 — Réplication (leader-follower)]]) n'a, seule, aucun mécanisme de failover automatique.

```bash
# sentinel.conf
sentinel monitor mon-cluster 192.168.1.10 6379 2
sentinel down-after-milliseconds mon-cluster 5000
sentinel failover-timeout mon-cluster 60000
```

Le dernier chiffre de `sentinel monitor` (`2` ci-dessus) est le **quorum** : le nombre de Sentinels devant s'accorder sur l'indisponibilité du leader avant de la considérer comme réelle.

## Deux votes distincts : détection puis autorisation

Le processus de failover se déroule en deux temps bien distincts :

1. **Détection (quorum)** : chaque Sentinel surveille le leader indépendamment. Dès qu'un Sentinel ne reçoit plus de réponse, il marque le leader `SDOWN` (*Subjectively Down*). Si au moins `quorum` Sentinels signalent la même panne, le leader passe en `ODOWN` (*Objectively Down*).
2. **Autorisation (majorité)** : une fois l'`ODOWN` constaté, les Sentinels élisent un Sentinel meneur, qui doit obtenir l'accord de la **majorité absolue** des Sentinels déployés (pas seulement le quorum) avant de procéder au failover.

> [!warning] Quorum et majorité ne sont pas le même chiffre
> Le quorum ne sert qu'à *détecter* la panne. Déclencher le failover exige toujours la majorité des Sentinels configurés, quel que soit le quorum choisi — avec 5 Sentinels et un quorum à 2, la détection peut se faire à 2, mais l'autorisation du failover nécessite l'accord de 3 Sentinels minimum.

## Le déroulement du failover

```bash
SENTINEL get-master-addr-by-name mon-cluster
```

1. Le Sentinel meneur sélectionne le follower le plus à jour parmi les répliques disponibles.
2. Il le promeut avec `REPLICAOF NO ONE`.
3. Il reconfigure les autres followers pour qu'ils répliquent depuis le nouveau leader.
4. Les clients interrogent Sentinel (pas directement une adresse fixe) pour connaître l'adresse courante du leader.

> [!tip] Les clients doivent connaître Sentinel, pas une IP fixe
> Une application doit se connecter via un client Redis compatible Sentinel (qui interroge `SENTINEL get-master-addr-by-name` pour retrouver le leader courant), plutôt que de coder en dur l'adresse du leader — sans quoi le failover automatique de Sentinel n'a aucun effet côté application.

## Nombre de Sentinels : 3 minimum, toujours impair

```
Sentinel 1 ─┐
Sentinel 2 ─┼─► surveillent le même leader/followers
Sentinel 3 ─┘
```

> [!warning] Deux Sentinels ne suffisent jamais à une majorité fiable
> Comme pour un cluster Galera (voir [[MariaDB 12 — Galera Cluster, réplication synchrone multi-maître]]), un nombre pair de Sentinels expose à une situation où aucune majorité claire ne peut se dégager après une coupure réseau. Déployer systématiquement un nombre impair de Sentinels, 3 au minimum, idéalement répartis sur des zones de panne distinctes.

## Pour aller plus loin

Sentinel gère la haute disponibilité d'un jeu de données qui tient sur une seule instance. Pour distribuer des données trop volumineuses pour un seul nœud, Redis Cluster répartit les clés sur plusieurs nœuds — détaillé dans [[Redis 13 — Cluster (sharding & hash slots)]].

Sources : [High availability with Redis Sentinel — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)
