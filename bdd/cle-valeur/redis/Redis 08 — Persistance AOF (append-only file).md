#bdd #redis #avancé

## Un journal de toutes les écritures

L'AOF (*Append Only File*) enregistre chaque commande qui modifie le jeu de données, dans l'ordre, au format du protocole Redis lui-même. Au redémarrage, Redis rejoue l'intégralité du journal pour reconstruire l'état exact des données.

```bash
# redis.conf
appendonly yes
appendfsync everysec
```

## Les trois politiques de fsync

| Politique | Comportement | Perte de données max |
|-----------|---------------|--------------------------|
| `always` | `fsync` à chaque écriture | Quasi nulle, mais très lent |
| `everysec` (défaut) | `fsync` une fois par seconde | ~1 seconde de données |
| `no` | Aucun `fsync` explicite, laissé au système d'exploitation | Dépend du noyau (souvent ~30 secondes) |

> [!tip] `everysec` est le compromis recommandé
> Rapide (le `fsync` s'exécute dans un thread d'arrière-plan, sans bloquer les écritures normales) et raisonnablement sûr (perte bornée à une seconde en cas de crash). `always` n'est justifié que pour des besoins de durabilité extrême, au prix d'un débit d'écriture nettement plus faible.

## Le format multi-fichiers depuis Redis 7.0

Depuis la version 7.0, l'AOF n'est plus un fichier unique mais un répertoire contenant un fichier de base (snapshot RDB ou AOF) et des fichiers incrémentaux, suivis par un fichier manifeste.

```bash
appenddirname "appendonlydir"
```

```
appendonlydir/
├── appendonly.aof.1.base.rdb       # État de référence
├── appendonly.aof.1.incr.aof       # Écritures depuis le dernier rewrite
└── appendonly.aof.manifest         # Suivi des fichiers actifs
```

## BGREWRITEAOF : compacter le journal

Un compteur incrémenté 100 fois génère 100 entrées dans l'AOF, alors qu'une seule valeur finale suffirait à reconstruire l'état. `BGREWRITEAOF` réécrit le journal avec la séquence minimale de commandes nécessaires pour recréer l'état courant :

```bash
BGREWRITEAOF
INFO persistence | grep aof_rewrite
```

> [!info] Un rewrite sans interruption de service
> Depuis Redis 7.0, le rewrite se déroule dans un processus enfant qui écrit un nouveau fichier de base, pendant que le processus parent continue à écrire dans un nouveau fichier incrémental. Les deux flux fusionnent ensuite via un remplacement atomique du manifeste — aucune donnée n'est perdue même si le rewrite échoue en cours de route.

## Réparer un AOF corrompu ou tronqué

```bash
redis-check-aof --fix appendonlydir/appendonly.aof.1.incr.aof
```

> [!warning] Un AOF tronqué reste généralement récupérable
> Si le serveur s'est arrêté brutalement en plein milieu d'une écriture, Redis peut charger l'AOF en ignorant simplement la dernière commande mal formée (`aof-load-truncated` activé par défaut). Une corruption au milieu du fichier (octets invalides, pas juste une troncature en fin de fichier) nécessite `redis-check-aof --fix`, avec un risque de perdre toutes les données après le point de corruption.

## RDB + AOF : le combo recommandé pour une durabilité proche de PostgreSQL

Activer les deux mécanismes ensemble donne un filet de sécurité en profondeur : l'AOF pour une durabilité fine (perte bornée à ~1 seconde), le RDB pour des sauvegardes compactes et un redémarrage rapide. Au redémarrage avec les deux actifs, Redis privilégie toujours l'AOF, garanti plus complet.

## Pour aller plus loin

Au-delà de la persistance, Redis offre aussi des garanties transactionnelles sur des séquences de commandes — voir [[Redis 09 — Transactions & scripting (MULTI, WATCH, Lua)]].

Sources : [Redis persistence — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
