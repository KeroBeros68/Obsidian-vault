#bdd #mysql #réplication #avancé

## Le failover consiste à promouvoir le replica

Le failover consiste à promouvoir le replica en nouveau source quand l'ancien source est indisponible.

> [!info] Le failover nécessite une reconfiguration
> Contrairement à PostgreSQL (`pg_promote()` en une commande), le failover MySQL nécessite plusieurs étapes : arrêter la réplication, désactiver le read-only, reconfigurer les applications. La procédure est plus manuelle, mais le GTID simplifie considérablement le repositionnement — voir [[MySQL 26 — Concepts de réplication & GTID]].

## Vérifier l'état avant le failover

Avant de promouvoir, s'assurer que le replica est synchronisé :

```sql
SHOW REPLICA STATUS\G
```

Vérifier :
- `Replica_IO_Running: Yes` et `Replica_SQL_Running: Yes`
- `Seconds_Behind_Source: 0`
- `Retrieved_Gtid_Set = Executed_Gtid_Set` (tous les GTID reçus sont appliqués)

Voir [[MySQL 28 — Surveiller la réplication & calculer le lag]] pour le détail de ces indicateurs.

## Promouvoir le replica

```sql
-- Arrêter la réplication
STOP REPLICA;

-- Supprimer la configuration de réplication
RESET REPLICA ALL;
```

> [!warning] `RESET REPLICA ALL` efface la configuration source
> Cette commande efface toute la configuration source (host, user, etc.) — le replica oublie qu'il était un replica. Avec GTID activé, `RESET REPLICA ALL` ne réinitialise **pas** l'historique GTID (`gtid_executed`, `gtid_purged`) : c'est le comportement souhaité pour le failover, le nouveau source conserve l'historique complet des transactions.

```sql
-- Désactiver le mode lecture seule
SET GLOBAL read_only = OFF;
SET GLOBAL super_read_only = OFF;
SET PERSIST read_only = OFF;
SET PERSIST super_read_only = OFF;
```

```sql
-- Vérifier que les écritures fonctionnent
INSERT INTO lab_mysql.clients (nom, email, ville)
VALUES ('Après failover', 'failover@example.com', 'Marseille');
-- Query OK, 1 row affected
```

Reconfigurer les applications pour pointer vers le nouveau source.

## Reconstruire l'ancien source comme nouveau replica

Une fois l'ancien source disponible à nouveau, le reconstituer comme replica du nouveau source :

```sql
-- Sur l'ancien source : réinitialiser les données via le clone plugin
SET GLOBAL clone_valid_donor_list = '192.168.122.72:3306';
CLONE INSTANCE FROM 'clone_user'@'192.168.122.72':3306
  IDENTIFIED BY 'motdepasse_clone';
```

```sql
-- Configurer la réplication vers le nouveau source
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST = '192.168.122.72',
  SOURCE_PORT = 3306,
  SOURCE_USER = 'replicator',
  SOURCE_PASSWORD = 'motdepasse_fort_replication',
  SOURCE_AUTO_POSITION = 1;

START REPLICA;
```

```sql
SHOW REPLICA STATUS\G
-- Replica_IO_Running: Yes
-- Replica_SQL_Running: Yes
```

Le clone plugin et l'alternative `mysqldump`/MySQL Shell sont détaillés dans [[MySQL 27 — Mettre en place une réplication source-replica (GTID)]].

## Pour aller plus loin

Le failover manuel devient automatique avec Group Replication — voir [[MySQL 30 — Semi-synchrone, Group Replication & InnoDB Cluster]].

Sources : [Réplication MySQL : source-replica, GTID et haute disponibilité — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/replication/)
