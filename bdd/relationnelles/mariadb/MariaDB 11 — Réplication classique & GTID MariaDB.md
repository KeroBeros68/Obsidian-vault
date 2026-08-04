#bdd #mariadb #réplication #avancé

## Un format GTID incompatible avec celui de MySQL

MariaDB implémente son propre format de GTID (*Global Transaction ID*), introduit en 10.0, structurellement différent du format `UUID:transaction_id` de MySQL (voir [[MySQL 26 — Concepts de réplication & GTID]]) :

```
0-1-345
```

Le format est `domaine-serveur-séquence` : `gtid_domain_id` (identifie un flux de réplication, utile en multi-source), l'ID du serveur qui a créé la transaction, puis un numéro de séquence incrémental.

> [!warning] Les deux formats GTID ne s'interopèrent pas
> Un GTID MariaDB (`0-1-345`) et un GTID MySQL (`3E11FA47-71CA-11E1-9E33-C80AA9429562:23`) ne sont pas convertibles l'un vers l'autre. Répliquer entre un MariaDB et un MySQL nécessite de revenir à la réplication par position de binlog (fichier + offset), sans les bénéfices du GTID d'aucun des deux côtés.

## gtid_domain_id : le multi-source natif

```sql
SET GLOBAL gtid_domain_id = 1;
SHOW VARIABLES LIKE 'gtid_domain_id';
```

Le `domain_id` permet à un même replica de recevoir plusieurs flux de réplication indépendants (plusieurs sources), chacun identifié par son propre domaine — un scénario que MariaDB gère nativement depuis l'introduction du GTID, sans la complexité additionnelle que ce cas impose côté MySQL.

## gtid_current_pos : la position courante

```sql
SELECT @@GLOBAL.gtid_current_pos;
```

`gtid_current_pos` est l'union de `gtid_binlog_pos` (GTID écrits dans le binlog local) et `gtid_slave_pos` (GTID reçus d'un source en tant que replica) — la référence à utiliser pour connaître l'état de réplication complet d'un nœud.

## Configurer un replica avec GTID

```sql
-- Sur le source : server_id unique, comme pour MySQL
-- /etc/mysql/mariadb.conf.d/50-server.cnf
-- [mysqld]
-- server_id = 1
-- log_bin = /var/lib/mysql/binlog
-- gtid_domain_id = 1

CREATE USER 'replicator'@'10.0.1.%' IDENTIFIED BY 'motdepasse_fort';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'10.0.1.%';
```

```sql
-- Sur le replica
CHANGE MASTER TO
  MASTER_HOST = '10.0.1.71',
  MASTER_USER = 'replicator',
  MASTER_PASSWORD = 'motdepasse_fort',
  MASTER_USE_GTID = slave_pos;

START SLAVE;
SHOW SLAVE STATUS\G
```

> [!info] Terminologie : MariaDB garde CHANGE MASTER TO et START SLAVE
> Contrairement à MySQL 8.0.22+, qui a renommé ses commandes en `CHANGE REPLICATION SOURCE TO`/`START REPLICA` (voir [[MySQL 27 — Mettre en place une réplication source-replica (GTID)]]), MariaDB conserve la terminologie historique `MASTER`/`SLAVE` dans sa syntaxe SQL — bien que la documentation utilise de plus en plus *source*/*replica* dans son vocabulaire général.

Les indicateurs à surveiller dans `SHOW SLAVE STATUS` restent conceptuellement identiques à ceux de MySQL : `Slave_IO_Running`, `Slave_SQL_Running`, `Seconds_Behind_Master`.

## Pour aller plus loin

Pour une haute disponibilité multi-maître avec failover automatique, MariaDB propose une solution que MySQL n'a pas : Galera Cluster, détaillé dans [[MariaDB 12 — Galera Cluster, réplication synchrone multi-maître]].

Sources : [Global Transaction ID — MariaDB Documentation](https://mariadb.com/kb/en/gtid/), [Enabling GTIDs for Server Replication in MariaDB Server — MariaDB](https://mariadb.com/resources/blog/enabling-gtids-for-server-replication-in-mariadb-server-10-2/)
