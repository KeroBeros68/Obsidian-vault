#bdd #mysql #réplication #avancé

## SHOW REPLICA STATUS : les colonnes clés

`SHOW REPLICA STATUS\G` contient plus de 60 colonnes. Voici les essentielles :

| Colonne | Signification |
|---------|-------------------|
| `Replica_IO_Running` | `Yes` = thread I/O actif (lecture binlog source) |
| `Replica_SQL_Running` | `Yes` = thread SQL actif (application relay log) |
| `Seconds_Behind_Source` | Lag en secondes (`0` = synchronisé) |
| `Retrieved_Gtid_Set` | GTID reçus du source |
| `Executed_Gtid_Set` | GTID appliqués sur le replica |
| `Last_IO_Error` | Dernière erreur du thread I/O |
| `Last_SQL_Error` | Dernière erreur du thread SQL |

## performance_schema.replication_*

Pour un monitoring plus structuré, utiliser les tables `performance_schema` (voir [[MySQL 16 — Performance Schema & sys schema]]) :

```sql
-- État des connexions de réplication
SELECT CHANNEL_NAME, SERVICE_STATE, SOURCE_UUID,
       LAST_ERROR_NUMBER, LAST_ERROR_MESSAGE
FROM performance_schema.replication_connection_status;

-- État des appliers (threads parallèles)
SELECT CHANNEL_NAME, WORKER_ID, SERVICE_STATE,
       LAST_APPLIED_TRANSACTION, APPLYING_TRANSACTION
FROM performance_schema.replication_applier_status_by_worker;
```

## Calculer le lag précisément

`Seconds_Behind_Source` est la métrique la plus courante mais elle peut être imprécise (elle mesure le delta entre le timestamp de l'événement et l'heure locale). Pour un calcul plus fiable, comparer les GTID exécutés vs reçus :

```sql
SELECT
  GTID_SUBTRACT(
    (SELECT Received_transaction_set
     FROM performance_schema.replication_connection_status),
    @@global.gtid_executed
  ) AS gtid_lag;
```

Si le résultat est vide, le replica a appliqué tous les GTID reçus : zéro lag.

## Multi-threaded applier (replica_parallel_workers)

En MySQL 8.4, les replicas sont multithreadés par défaut : `replica_parallel_workers` vaut `4`, avec `replica_preserve_commit_order = ON`.

```sql
STOP REPLICA;
SET PERSIST replica_parallel_workers = 8;
SET PERSIST replica_preserve_commit_order = ON;
START REPLICA;
```

| Paramètre | Défaut 8.4 | Rôle |
|-----------|-----------|------|
| `replica_parallel_workers` | `4` | Nombre de threads d'application (`0` = mono-thread) |
| `replica_parallel_type` | `LOGICAL_CLOCK` | Parallélise par groupe de transactions commitées en même temps sur le source |
| `replica_preserve_commit_order` | `ON` | Garantit que l'ordre des commits est respecté même en parallèle |

## Tester la réplication en temps réel

```sql
-- Sur le source
INSERT INTO lab_mysql.clients (nom, email, ville)
VALUES ('Test Replication', 'repl@example.com', 'Lyon');

-- Sur le replica, immédiatement
SELECT nom, email FROM lab_mysql.clients WHERE nom = 'Test Replication';
```

La ligne est visible quasi instantanément.

## read_only et super_read_only

Le replica est configuré en lecture seule via deux paramètres :

```sql
SHOW VARIABLES LIKE '%read_only%';
-- read_only = ON, super_read_only = ON
```

`read_only = ON` bloque les écritures pour les utilisateurs normaux. `super_read_only = ON` bloque aussi les écritures pour les comptes disposant de `SUPER` ou `CONNECTION_ADMIN` — recommandé pour éviter les écritures accidentelles par un DBA.

```sql
INSERT INTO lab_mysql.clients (nom, email, ville) VALUES ('Test', 'test@example.com', 'Paris');
-- ERROR 1290 (HY000): The MySQL server is running with the
-- --super-read-only option so it cannot execute this statement
```

## Connecter les applications en lecture

Pour distribuer les lectures : au niveau applicatif (deux connexions, source pour l'écriture et replica pour la lecture), ou via un proxy (ProxySQL ou MySQL Router font le routage automatiquement selon le type de requête — voir [[MySQL 30 — Semi-synchrone, Group Replication & InnoDB Cluster]]).

> [!warning] Attention au lag
> Juste après une écriture sur le source, la donnée peut ne pas être encore visible sur le replica (lag asynchrone). Pour les lectures qui doivent voir les données fraîches (« *read-your-writes* »), lire depuis le source.

## Pour aller plus loin

Quand le source devient indisponible, le failover manuel est détaillé dans [[MySQL 29 — Failover manuel & reconstruction]].

Sources : [Réplication MySQL : source-replica, GTID et haute disponibilité — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/replication/)
