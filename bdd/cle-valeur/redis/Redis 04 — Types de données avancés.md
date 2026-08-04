#bdd #redis #avancé

## Streams : un journal d'événements append-only

Un Stream stocke une séquence d'entrées horodatées, chacune composée de paires champ-valeur — conçu pour l'ajout et la lecture ordonnée, avec un support natif des **groupes de consommateurs** garantissant une livraison *at-least-once*.

```bash
XADD evenements '*' type "connexion" utilisateur "42"
XLEN evenements
XRANGE evenements - +                       # Toutes les entrées

XGROUP CREATE evenements groupe_traitement '$'
XREADGROUP GROUP groupe_traitement worker1 COUNT 10 STREAMS evenements '>'
XACK evenements groupe_traitement <id_entree>
```

> [!info] Streams vs List : la vraie différence
> Une List Redis n'offre pas de suivi de position pour plusieurs lecteurs indépendants — une fois `LPOP` exécuté, l'élément a disparu pour tout le monde. Un Stream conserve son historique et permet à plusieurs groupes de consommateurs de lire indépendamment, chacun à son propre rythme, avec accusé de réception explicite (`XACK`) — un modèle proche de Kafka, en plus léger.

## Bitmaps : des opérations bit à bit sur des chaînes

Un Bitmap n'est pas un type distinct : c'est une String manipulée au niveau du bit, idéale pour des indicateurs booléens massifs et compacts.

```bash
SETBIT utilisateurs_actifs:2026-07-28 42 1   # L'utilisateur 42 est actif aujourd'hui
GETBIT utilisateurs_actifs:2026-07-28 42
BITCOUNT utilisateurs_actifs:2026-07-28        # Nombre total de bits à 1
BITOP AND resultat utilisateurs_actifs:2026-07-27 utilisateurs_actifs:2026-07-28  # Actifs les deux jours
```

> [!tip] Cas d'usage type : suivi d'activité quotidienne
> Un bit par utilisateur et par jour permet de calculer des cohortes (utilisateurs actifs 7 jours de suite, rétention) avec `BITOP`/`BITCOUNT`, pour une fraction de la mémoire qu'un Set d'identifiants utilisateur occuperait.

## HyperLogLog : compter des éléments uniques approximativement

```bash
PFADD visiteurs_uniques "ip1" "ip2" "ip3"
PFCOUNT visiteurs_uniques   # Estimation du nombre d'éléments uniques
```

Une structure probabiliste qui estime la cardinalité d'un ensemble (nombre d'éléments distincts) avec une erreur standard d'environ 0,81 %, en n'utilisant que quelques kilo-octets — quel que soit le nombre réel d'éléments ajoutés (des milliers ou des milliards).

> [!warning] Une approximation, pas un compte exact
> HyperLogLog convient au comptage de visiteurs uniques ou de vues distinctes à grande échelle, où une petite erreur est acceptable. Pour un besoin de cardinalité exacte sur un petit volume, un Set classique (`SADD`/`SCARD`) reste préférable.

## Géospatial : coordonnées et recherche de proximité

```bash
GEOADD lieux -0.1276 51.5072 "Londres" 2.3522 48.8566 "Paris"
GEODIST lieux Londres Paris km
GEOSEARCH lieux FROMMEMBER Paris BYRADIUS 500 km ASC
```

Implémenté au-dessus d'un Sorted Set (les coordonnées sont encodées en *geohash* comme score), le type géospatial permet des recherches de rayon et de distance sans structure ni index séparés.

## Redis 8 : des types avancés désormais intégrés

Depuis Redis 8.0, les modules autrefois vendus séparément sous *Redis Stack* (JSON natif, recherche full-text avec RediSearch, séries temporelles, filtres probabilistes Bloom/Cuckoo, `top-k`, `count-min sketch`, `t-digest`) sont inclus nativement dans Redis Open Source, sous la même licence tri-choix que le cœur du serveur.

## Pour aller plus loin

La gestion du cycle de vie des clés — expiration et suppression sous pression mémoire — est couverte dans [[Redis 05 — Expiration des clés (TTL)]].

Sources : [Redis data types — Redis Documentation](https://redis.io/docs/latest/develop/data-types/), [Redis Open Source 8.0 release notes — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/release-notes/redisce/redisos-8.0-release-notes/)
