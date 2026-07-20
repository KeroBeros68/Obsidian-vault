#bdd #mysql #monitoring #intermédiaire

## Performance Schema : l'instrumentation interne

**Performance Schema** collecte des statistiques sur pratiquement tout : requêtes, verrous, I/O, mémoire, threads, étapes internes. Activé par défaut depuis MySQL 5.6.

```sql
SHOW VARIABLES LIKE 'performance_schema';   -- ON
SHOW ENGINE PERFORMANCE_SCHEMA STATUS;      -- consommation mémoire réelle
```

Il fonctionne en deux couches :

- **Instruments** : les points de mesure eux-mêmes (requêtes, verrous, I/O, mémoire...) — environ 1200 disponibles.
- **Consumers** : les tables qui stockent effectivement les données collectées par ces instruments.

```sql
SELECT NAME, ENABLED, TIMED FROM performance_schema.setup_instruments
WHERE NAME LIKE 'statement/sql/%' LIMIT 5;
```

## Les tables events_statements_*

| Table | Contenu |
|-------|---------|
| `events_statements_current` | Requêtes en cours d'exécution |
| `events_statements_history` | Dernières requêtes par thread |
| `events_statements_history_long` | Historique étendu (configurable) |
| `events_statements_summary_by_digest` | Statistiques agrégées par type de requête (normalisation) |
| `events_statements_summary_by_user_by_event_name` | Statistiques agrégées par utilisateur |

Ce sont ces tables brutes que le **sys schema** (installé par défaut depuis MySQL 5.7) transforme en vues directement exploitables.

## sys.statement_analysis : le top des requêtes coûteuses

```sql
SELECT query, exec_count, avg_latency, total_latency, rows_sent_avg, rows_examined_avg, full_scan
FROM sys.statement_analysis LIMIT 10;
```

Équivalent MySQL du top `pg_stat_statements` de PostgreSQL, prêt à l'emploi sans installation.

| Signal | Ce qu'il révèle |
|--------|----------------------|
| `total_latency` élevé | La requête qui consomme le plus de temps serveur au total — priorité n°1 |
| `avg_latency` élevé + `exec_count` bas | Requête unitairement lente, index manquant probable |
| `full_scan = *` | Full table scan — index manquant |
| `rows_examined_avg` >> `rows_sent_avg` | MySQL lit beaucoup trop de lignes pour en retourner peu |

## Autres vues sys utiles

```sql
-- Tables occupant le plus le buffer pool
SELECT object_schema, object_name, allocated, data, pages
FROM sys.innodb_buffer_stats_by_table
WHERE object_schema NOT IN ('mysql', 'sys', 'performance_schema') LIMIT 5;
```

> [!warning] `innodb_buffer_stats_by_table` est coûteuse à interroger
> Elle s'appuie sur `INFORMATION_SCHEMA.INNODB_BUFFER_PAGE`, dont l'interrogation a un impact mesurable sur les performances — à éviter en production sauf besoin ponctuel et maîtrisé.

```sql
-- Alternative à SHOW PROCESSLIST, avec latence de transaction/verrou en plus
SELECT thd_id, conn_id, user, db, command, state, time,
       LEFT(current_statement, 60) AS query, trx_latency, lock_latency
FROM sys.processlist WHERE conn_id IS NOT NULL;

-- Index redondants (un index préfixe d'un autre)
SELECT table_schema, table_name, redundant_index_name, redundant_index_columns,
       dominant_index_name, dominant_index_columns
FROM sys.schema_redundant_indexes WHERE table_schema = 'lab_mysql';

-- Clients/utilisateurs les plus actifs
SELECT host, statements, total_latency, rows_examined, full_scans FROM sys.host_summary;
SELECT user, statements, total_latency, rows_examined, full_scans FROM sys.user_summary;
```

> [!tip] Un index redondant coûte sans bénéfice
> Un index qui n'est qu'un préfixe d'un autre index consomme de l'espace et ralentit les écritures (`INSERT`/`UPDATE`/`DELETE`) sans apporter le moindre bénéfice aux lectures — à supprimer après vérification via `sys.schema_redundant_indexes`.

## Cas particuliers

> [!info] `sys` n'ajoute aucune donnée, seulement de la lisibilité
> Rappel (voir [[MySQL 02 — Bases de données, schémas & bases système]]) : tout ce qu'expose `sys` existe déjà dans `performance_schema`/`information_schema`, sous une forme moins directement exploitable.

## Pour aller plus loin

Une fois une requête coûteuse identifiée via `sys.statement_analysis`, `EXPLAIN`/`EXPLAIN ANALYZE` permet de comprendre précisément pourquoi — voir [[MySQL 17 — EXPLAIN & EXPLAIN ANALYZE]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
