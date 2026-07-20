#bdd #mysql #monitoring #avancé

## mysqld_exporter : exposer MySQL à Prometheus

```sql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'motdepasse_exporter' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
```

```ini
# ~/.my.cnf pour l'exporter
[client]
user=exporter
password=motdepasse_exporter
```

| Métrique Prometheus | Source MySQL | Seuil d'alerte |
|--------------------------|-------------------|---------------------|
| `mysql_global_status_threads_connected` | `Threads_connected` | > 80% de `max_connections` |
| `mysql_global_status_slow_queries` | `Slow_queries` | Augmentation soudaine |
| `mysql_global_status_innodb_buffer_pool_reads` | `Innodb_buffer_pool_reads` | Hit ratio < 95% (voir [[MySQL 14 — SHOW GLOBAL STATUS & SHOW ENGINE INNODB STATUS]]) |
| `mysql_global_status_innodb_row_lock_waits` | `Innodb_row_lock_waits` | > 0 de manière soutenue |
| `mysql_slave_status_seconds_behind_master` | `Seconds_Behind_Source` | > 30 secondes |

> [!tip] Dashboard Grafana de référence
> **MySQL Overview** (ID `7362` sur grafana.com) est le dashboard le plus populaire — buffer pool hit ratio, connexions, QPS, lag de réplication et métriques InnoDB en un seul écran.

## Alertes critiques à configurer

| Alerte | Condition | Criticité |
|--------|---------------|---------------|
| Connexions proches de la limite | `Threads_connected` > 80% `max_connections` | Critique |
| Buffer pool hit ratio faible | < 95% pendant 5 min | Warning |
| Lag de réplication | `Seconds_Behind_Source` > 60 s | Critique |
| Réplication cassée | `Replica_IO_Running = No` ou `Replica_SQL_Running = No` | Critique |
| Slow queries en hausse | +50% par rapport à la baseline | Warning |
| Espace disque | < 20% libre sur le datadir | Critique |
| Deadlocks fréquents | > 10 par heure | Warning |

> [!warning] La réplication cassée est l'alerte la plus importante à ne jamais manquer
> Toute minute sans réplication active est une minute de données non protégées côté réplica — cette alerte mérite une priorité supérieure à la plupart des autres métriques de performance.

## Tableau de dépannage récapitulatif

| Symptôme | Cause probable | Solution |
|----------|--------------------|----------|
| Requête bloquée par un lock | Transaction longue non commitée | `SELECT * FROM sys.innodb_lock_waits\G`, tuer la transaction bloquante (voir [[MySQL 13 — Observer l'activité (PROCESSLIST & connexions)]]) |
| Buffer pool hit ratio < 95% | `innodb_buffer_pool_size` trop petit | Augmenter à 50-70% de la RAM disponible |
| Slow queries en hausse soudaine | Index supprimé ou statistiques obsolètes | `ANALYZE TABLE`, vérifier `sys.schema_redundant_indexes` |
| "Too many connections" | Pool non fermé ou `wait_timeout` trop élevé | Réduire `wait_timeout`, utiliser un connection pooler (ProxySQL) |
| Table qui grossit après DELETE/UPDATE | Espace non récupéré au niveau fichier | `OPTIMIZE TABLE` (voir [[MySQL 19 — Maintenance des tables]]) |
| `EXPLAIN` montre `ALL` sur une grosse table | Index manquant sur les colonnes filtrées | Ajouter un index composite sur les colonnes du `WHERE` |
| `History list length` qui augmente | Thread de purge en retard, `SELECT` très longs | Identifier les transactions longues, augmenter `innodb_purge_threads` |
| Deadlocks fréquents | Ordre d'accès aux tables/lignes incohérent | Accéder aux tables toujours dans le même ordre, transactions courtes, `innodb_print_all_deadlocks = ON` |

## Cas particuliers

> [!info] Ce guide couvre l'observation, pas encore la correction structurelle
> Le dimensionnement fin (buffer pool, connexions, binary log) reste couvert par un futur guide de configuration ; la sauvegarde/restauration et la réplication (GTID, semi-synchrone, Group Replication) restent également non couvertes dans ce vault pour l'instant — voir [[MySQL — Index des fiches]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
