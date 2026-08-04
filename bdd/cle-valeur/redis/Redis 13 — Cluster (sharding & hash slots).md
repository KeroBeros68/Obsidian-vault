#bdd #redis #réplication #avancé

## Distribuer les données sur plusieurs nœuds

Redis Cluster répond à un besoin différent de Sentinel : là où Sentinel garantit la disponibilité d'un jeu de données qui tient sur une seule instance, Cluster **partitionne** les données (sharding) entre plusieurs nœuds maîtres, chacun pouvant avoir ses propres followers pour la haute disponibilité.

## 16384 hash slots : le mécanisme de répartition

L'espace des clés est divisé en **16384 hash slots**. Chaque clé est assignée à un slot par `CRC16(clé) mod 16384`, et chaque nœud maître du cluster prend en charge un sous-ensemble de ces slots.

```bash
CLUSTER ADDSLOTS 0 1 2 3 ... 5460    # Nœud A : slots 0-5460
# Nœud B : slots 5461-10922
# Nœud C : slots 10923-16383

CLUSTER KEYSLOT ma_cle                # Quel slot pour une clé donnée
CLUSTER NODES                         # Topologie complète du cluster
```

> [!info] Pourquoi 16384 précisément
> Ce nombre est un compromis délibéré : assez petit pour que la carte des slots tienne dans un message de heartbeat compact (2 Ko pour 16384 bits), assez grand pour répartir finement les données sur plusieurs centaines de nœuds sans grain trop grossier.

## Hash tags : forcer plusieurs clés sur le même slot

```bash
SET utilisateur:{42}:profil "..."
SET utilisateur:{42}:preferences "..."
-- Les deux clés partagent le même hash slot grâce à {42}
```

Seule la portion entre accolades `{}` est utilisée pour le calcul du slot. Ce mécanisme est indispensable pour toute opération multi-clés (`MGET`, transaction `MULTI`, script Lua) portant sur plusieurs clés : Redis Cluster refuse une opération multi-clés dont les clés sont réparties sur des slots différents.

> [!warning] Sans hash tag, une opération multi-clés peut échouer
> `MGET cle1 cle2` échoue avec une erreur `CROSSSLOT` si `cle1` et `cle2` finissent sur des slots différents. Concevoir le nommage des clés avec des hash tags dès la conception, pour tout ensemble de clés appelé à être manipulé ensemble.

## Le protocole gossip : chaque nœud connaît l'état du cluster

Les nœuds échangent en permanence des messages de battement de cœur (*heartbeat*) en topologie maillée (chaque nœud avec chaque autre), propageant l'état de santé des pairs et la carte des slots — sans dépendre d'un nœud central de coordination.

```bash
CLUSTER INFO
-- cluster_state:ok / fail
```

## Redirection côté client : MOVED et ASK

Un client peut interroger n'importe quel nœud du cluster. Si la clé demandée appartient à un autre nœud, celui-ci répond `MOVED <slot> <adresse>` plutôt que de faire suivre la requête lui-même — le client doit alors réinterroger directement le bon nœud. Les clients Redis modernes maintiennent en cache la carte des slots pour éviter ce détour à chaque requête.

```bash
GET ma_cle
-- (error) MOVED 12345 192.168.1.12:6379
```

> [!info] ASK : une redirection temporaire pendant un resharding
> Pendant un déplacement de slot en cours (`CLUSTER RESHARD`), le nœud source répond `ASK` (plutôt que `MOVED`) pour les clés déjà déplacées — signalant au client qu'il s'agit d'une redirection ponctuelle, pas d'un changement permanent de la carte des slots.

## Prérequis et compromis

- **6 nœuds minimum en production** (3 maîtres + 3 followers) pour combiner sharding et haute disponibilité par maître.
- Pas de sélection de base (`SELECT n`) : Redis Cluster n'expose que la base `0`.
- Toute opération multi-clés doit respecter les hash tags pour rester sur un seul slot.

## Pour aller plus loin

Que ce soit en instance unique, avec Sentinel ou en Cluster, sécuriser l'accès reste indispensable — voir [[Redis 14 — Sécurité (ACL, TLS & durcissement)]].

Sources : [Redis cluster specification — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)
