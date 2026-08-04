#bdd #redis #avancé

## maxmemory : la limite mémoire du cache

```bash
CONFIG SET maxmemory 100mb
CONFIG SET maxmemory-policy allkeys-lru
```

Quand la mémoire utilisée dépasse `maxmemory`, Redis applique la politique d'éviction configurée pour libérer de l'espace avant d'accepter une nouvelle écriture.

## Les politiques disponibles

| Politique | Comportement |
|-----------|--------------|
| `noeviction` | Refuse les écritures avec une erreur une fois la limite atteinte — les lectures restent possibles |
| `allkeys-lru` | Évince les clés les moins récemment utilisées (accès en lecture ou écriture), parmi toutes les clés |
| `volatile-lru` | Idem, mais uniquement parmi les clés ayant un TTL configuré |
| `allkeys-lfu` | Évince les clés les moins fréquemment utilisées, parmi toutes les clés |
| `volatile-lfu` | Idem, mais uniquement parmi les clés à TTL |
| `allkeys-lrm` | *(Redis 8.6+)* Évince les clés les moins récemment **modifiées** (écriture uniquement) |
| `volatile-lrm` | Idem, parmi les clés à TTL |
| `allkeys-random` | Évince des clés au hasard, parmi toutes les clés |
| `volatile-random` | Idem, parmi les clés à TTL |
| `volatile-ttl` | Évince en priorité les clés dont le TTL restant est le plus court |

> [!tip] `allkeys-lru` est le choix par défaut raisonnable
> Conforme au principe de Pareto (une minorité de clés concentre la majorité des accès), `allkeys-lru` convient à la plupart des usages de cache sans configuration plus fine. `volatile-ttl` est pertinent quand l'application sait déjà quelles clés sont de bonnes candidates à l'éviction et leur assigne un TTL court en conséquence.

> [!info] LRM : la nouveauté de Redis 8.6
> *Least Recently Modified* ne met à jour son horodatage qu'en écriture, contrairement à LRU qui le met à jour aussi en lecture. Utile quand une charge est très majoritairement en lecture : LRM évince les données réellement stagnantes plutôt que des données très lues mais jamais modifiées.

## LRU et LFU sont approximés, pas exacts

Redis n'implémente pas un LRU strict (qui nécessiterait une structure de données coûteuse à maintenir pour chaque accès). Il échantillonne un petit nombre de clés au hasard et évince celles dont l'accès est le plus ancien parmi l'échantillon :

```bash
CONFIG SET maxmemory-samples 10   # Défaut : 5 — plus élevé = plus proche du LRU exact, coût CPU accru
```

Pour le mode LFU, la fréquence d'accès est approximée par un **compteur de Morris** (compteur probabiliste sur quelques bits par clé, avec décroissance dans le temps pour s'adapter à un changement de pattern d'accès) :

```bash
CONFIG SET lfu-log-factor 10   # Vitesse de saturation du compteur
CONFIG SET lfu-decay-time 1    # Décroissance en minutes
```

## Surveiller l'efficacité du cache

```bash
INFO stats
-- keyspace_hits, keyspace_misses, evicted_keys, expired_keys
```

```
Taux de hit = keyspace_hits / (keyspace_hits + keyspace_misses) * 100
```

> [!warning] Un taux de hit bas ne signifie pas toujours « mauvaise politique »
> Vérifier `evicted_keys` : un nombre élevé indique que la politique évince trop souvent les mauvaises clés (envisager `allkeys-lru` si ce n'est pas déjà le cas). Un `evicted_keys` bas mais un taux de hit décevant pointe plutôt vers un TTL mal choisi (`expired_keys` élevé) ou une taille de cache (`maxmemory`) insuffisante pour le pattern d'accès réel.

## Pour aller plus loin

Un mécanisme d'éviction ne remplace pas une stratégie de durabilité : la persistance sur disque via RDB est détaillée dans [[Redis 07 — Persistance RDB (snapshots)]].

Sources : [Key eviction — Redis Documentation](https://redis.io/docs/latest/develop/reference/eviction/)
