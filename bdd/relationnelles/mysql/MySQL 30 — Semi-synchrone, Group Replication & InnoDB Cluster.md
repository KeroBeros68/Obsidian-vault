#bdd #mysql #réplication #avancé

## Réplication semi-synchrone : le COMMIT attend l'ACK du replica

En mode semi-synchrone, le source ne retourne le `COMMIT` au client que lorsqu'au moins un replica a confirmé avoir reçu et écrit le binlog event dans son relay log. Cela garantit que les données existent sur au moins deux serveurs avant la confirmation : RPO quasi nul.

```sql
-- Sur le source
INSTALL PLUGIN rpl_semi_sync_source SONAME 'semisync_source.so';
SET GLOBAL rpl_semi_sync_source_enabled = ON;
SET PERSIST rpl_semi_sync_source_enabled = ON;
```

```sql
-- Sur le replica
INSTALL PLUGIN rpl_semi_sync_replica SONAME 'semisync_replica.so';
SET GLOBAL rpl_semi_sync_replica_enabled = ON;
SET PERSIST rpl_semi_sync_replica_enabled = ON;

-- Redémarrer le thread I/O pour activer le semi-sync
STOP REPLICA IO_THREAD;
START REPLICA IO_THREAD;
```

```sql
-- Vérifier sur le source
SHOW STATUS LIKE 'Rpl_semi_sync_source_status';  -- ON
```

| Paramètre | Défaut | Rôle |
|-----------|--------|------|
| `rpl_semi_sync_source_timeout` | 10000 ms | Temps d'attente avant fallback en asynchrone |
| `rpl_semi_sync_source_wait_for_replica_count` | 1 | Nombre de replicas devant confirmer |
| `rpl_semi_sync_source_wait_point` | `AFTER_SYNC` | Le `COMMIT` attend après l'écriture dans le binlog (avant le commit InnoDB) — *loss-less semi-sync* |

Si aucun replica ne confirme dans le timeout, le source bascule automatiquement en mode asynchrone pour ne pas bloquer les écritures. Il repasse en semi-synchrone dès qu'un replica se reconnecte :

```sql
SHOW STATUS LIKE 'Rpl_semi_sync_source_no_tx';
SHOW STATUS LIKE 'Rpl_semi_sync_source_yes_tx';
```

> [!warning] Un seul replica semi-synchrone = risque
> Si votre unique replica tombe, le source passe en asynchrone silencieusement : les transactions ne sont plus répliquées de manière synchrone, la garantie RPO=0 est perdue sans notification explicite. En production, utiliser au moins deux replicas semi-synchrones.

## Group Replication : aperçu

Group Replication (GR) est la solution de haute disponibilité intégrée à MySQL. Elle fournit une réplication de machine à états distribuée avec forte coordination entre nœuds — le moteur de communication du groupe, XCom, est une variante de Paxos. Détection de pannes et failover automatiques.

| Mode | Description | Cas d'usage |
|------|--------------|--------------|
| Single-primary | Un seul nœud accepte les écritures, les autres sont en lecture | Le plus courant, le plus simple, évite les conflits |
| Multi-primary | Tous les nœuds acceptent les écritures | Requiert une gestion des conflits au niveau applicatif |

Group Replication impose des contraintes strictes :
- InnoDB uniquement, toutes les tables doivent utiliser InnoDB
- Clé primaire sur chaque table (pas de table sans PK)
- GTID activé (`gtid_mode = ON`, `enforce_gtid_consistency = ON`)
- Binary log en format `ROW`
- 3 nœuds minimum (nombre impair recommandé pour le quorum)
- Réseau fiable entre les nœuds (latence faible)

> [!info] `group_replication_consistency` en 8.4
> MySQL 8.4 change la valeur par défaut de `group_replication_consistency` de `EVENTUAL` (en 8.0) à `BEFORE_ON_PRIMARY_FAILOVER`, une amélioration significative de la cohérence après failover.

| Valeur | Comportement |
|--------|---------------|
| `EVENTUAL` | Les lectures peuvent voir des données légèrement en retard |
| `BEFORE_ON_PRIMARY_FAILOVER` | Défaut 8.4 — après un failover, le nouveau primary attend que les transactions en attente soient appliquées avant d'accepter les lectures |
| `BEFORE` | Chaque lecture attend que toutes les transactions du groupe soient appliquées |
| `AFTER` | Le `COMMIT` attend que tous les nœuds aient appliqué la transaction |
| `BEFORE_AND_AFTER` | Combine `BEFORE` et `AFTER`, cohérence maximale au prix de la latence |

```sql
SET PERSIST group_replication_consistency = 'BEFORE_ON_PRIMARY_FAILOVER';
```

## InnoDB Cluster et InnoDB ReplicaSet : orchestration

InnoDB Cluster combine trois composants : **Group Replication** (moteur de réplication sous-jacent), **MySQL Shell** (`mysqlsh`, outil d'administration pour créer et gérer le cluster) et **MySQL Router** (proxy qui route les connexions vers le primary pour les écritures et les secondaries pour les lectures).

```
Application
     │
     ▼
MySQL Router (port 6446 R/W, port 6447 R/O)
     │
     ├── Node 1 (Primary)   ← écritures
     ├── Node 2 (Secondary) ← lectures
     └── Node 3 (Secondary) ← lectures
```

```javascript
// Connecté au premier nœud
var cluster = dba.createCluster('monCluster');
cluster.addInstance('root@192.168.122.72:3306');
cluster.addInstance('root@192.168.122.73:3306');
cluster.status();
```

Si Group Replication est trop contraignant, **InnoDB ReplicaSet** offre une orchestration simplifiée pour la réplication asynchrone classique via MySQL Shell :

```javascript
var rs = dba.createReplicaSet('monReplicaSet');
rs.addInstance('root@192.168.122.72:3306');
rs.status();

// Failover
rs.setPrimaryInstance('root@192.168.122.72:3306');
```

| Critère | InnoDB Cluster | InnoDB ReplicaSet |
|---------|----------------|----------------------|
| Réplication | Group Replication (consensus) | Asynchrone classique |
| Failover | Automatique | Semi-automatique (via `mysqlsh`) |
| Nombre de nœuds | 3+ (impair) | 2+ |
| Contraintes | PK obligatoire, InnoDB only | Moins contraignant |
| Complexité | Élevée | Modérée |
| Cas d'usage | HA critique en production | Réplication simple avec orchestration |

## Outils tiers

**Orchestrator** (GitHub) : outil open source pour la gestion et le failover automatique des topologies de réplication MySQL — détecte les pannes, réarrange la topologie et effectue le failover sans intervention humaine.

**ProxySQL** : proxy MySQL haute performance qui gère le routage lecture/écriture, le connection pooling, le query caching et le failover automatique — alternative populaire à MySQL Router.

**MySQL Router** : proxy officiel Oracle, optimisé pour InnoDB Cluster. Route les connexions selon le rôle (R/W → primary, R/O → secondaries) et met à jour sa configuration automatiquement quand la topologie change.

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| `Replica_IO_Running: No` | Connectivité réseau, credentials incorrects ou `server_id` dupliqué | Vérifier avec `mysql -h source -u replicator -p`, vérifier `server_id` unique |
| `Got fatal error from source when reading data` | Binlog purgé avant que le replica ne l'ait lu | Augmenter `binlog_expire_logs_seconds` ou reconstruire le replica (clone/dump) |
| `Seconds_Behind_Source` augmente | Le replica n'arrive pas à suivre le rythme | Activer `replica_parallel_workers`, vérifier les IOPS disque du replica |
| `Duplicate entry for key 'PRIMARY'` | Transaction déjà appliquée (GTID manquant) | Avec GTID, le doublon est normalement évité — vérifier `gtid_executed` sur les deux nœuds |
| `Could not find first log file name in binary log index` | Le replica demande un binlog qui a été purgé | Reconstruire le replica avec le clone plugin ou un dump frais |
| GTID errant (*errant transaction*) | Transaction exécutée directement sur le replica | Identifier avec `GTID_SUBTRACT(replica_gtid, source_gtid)`. Injecter une transaction vide sur le source pour combler, ou reconstruire le replica |
| Semi-sync timeout constant | Réseau lent ou replica surchargé | Vérifier la latence réseau, augmenter `rpl_semi_sync_source_timeout` temporairement, vérifier les IOPS |
| `ERROR 3100 (HY000): The function is not allowed when Group Replication is running` | Commande incompatible avec Group Replication | Utiliser les APIs MySQL Shell (`dba.getCluster()`) au lieu des commandes SQL directes |

## À retenir

- La réplication GTID est le standard en MySQL 8.4 — elle simplifie le failover en identifiant chaque transaction de manière unique.
- La réplication n'est pas une sauvegarde : un `DROP TABLE` est immédiatement répliqué.
- Le clone plugin simplifie l'initialisation du replica en copiant le data directory complet — l'équivalent MySQL de `pg_basebackup`.
- `CHANGE REPLICATION SOURCE TO` avec `SOURCE_AUTO_POSITION = 1` exploite les GTID, pas besoin de calculer la position binlog manuellement.
- `read_only` + `super_read_only` protègent le replica contre les écritures accidentelles — activer toujours les deux.
- Le multi-threaded applier (`replica_parallel_workers`) accélère significativement le rattrapage du replica.
- La réplication semi-synchrone garantit un RPO quasi nul mais ajoute de la latence — avec un seul replica, le fallback en asynchrone est silencieux.
- Group Replication fournit le failover automatique par consensus, impose des contraintes strictes (InnoDB, clé primaire, 3+ nœuds).
- InnoDB Cluster = Group Replication + MySQL Shell + MySQL Router, la solution HA complète d'Oracle.
- En production, utiliser un outil d'orchestration (MySQL Router, ProxySQL, Orchestrator) plutôt qu'un failover manuel.

## Pour aller plus loin

Ceci conclut le module MySQL — voir [[MySQL — Index des fiches]] pour une vue d'ensemble des 30 fiches, ou [[BDD — Home]] pour explorer les autres moteurs de bases de données.

Sources : [Réplication MySQL : source-replica, GTID et haute disponibilité — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/replication/)
