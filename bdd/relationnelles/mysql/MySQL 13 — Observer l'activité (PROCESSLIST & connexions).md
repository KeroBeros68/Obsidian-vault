#bdd #mysql #monitoring #intermédiaire

## La démarche : observer → identifier → expliquer → corriger

Diagnostiquer un problème de performance MySQL suit toujours le même ordre : **observer** ce qui se passe en ce moment, **identifier** les requêtes les plus coûteuses, **expliquer** pourquoi elles sont lentes, puis **corriger**.

> [!warning] Ne jamais sauter d'étape
> Un `OPTIMIZE TABLE` lancé à l'aveugle sur une table de production verrouille tout en écriture avec l'algorithme par défaut (voir [[MySQL 19 — Maintenance des tables]]). Un index ajouté sans avoir lu le plan d'exécution peut aggraver la situation plutôt que la résoudre.

## SHOW PROCESSLIST : le premier réflexe

```sql
SHOW PROCESSLIST;
```

| Colonne | Signification |
|---------|-------------------|
| `Id` | Identifiant de connexion, utilisable avec `KILL` |
| `Command` | Type d'opération (`Query`, `Sleep`, `Connect`, `Daemon`) |
| `Time` | Secondes depuis le début de l'état courant |
| `State` | Étape interne (*sending data*, *executing*, *Waiting for lock*...) |
| `Info` | Requête en cours d'exécution |

> [!warning] `SHOW PROCESSLIST` tronque `Info` à 100 caractères
> Pour voir la requête complète, interroger directement la vue Performance Schema :
> ```sql
> SELECT ID, USER, HOST, DB, COMMAND, TIME AS time_sec, STATE, LEFT(INFO, 80) AS query
> FROM performance_schema.processlist WHERE COMMAND <> 'Daemon';
> ```

## Les connexions Sleep : le problème le plus courant

```sql
SELECT COMMAND, COUNT(*) AS cnt FROM performance_schema.processlist GROUP BY COMMAND;
```

| Command | Signification | Action si trop nombreux |
|---------|-------------------|------------------------------|
| `Query` | Exécute une requête | Normal, vérifier si trop long |
| `Sleep` | Connecté, ne fait rien | Pool de connexions mal configuré ou `wait_timeout` trop élevé |
| `Daemon` | Processus interne (event_scheduler) | Normal |

Une application qui ouvre des connexions sans les fermer remplit `max_connections` de connexions `Sleep`.

```sql
-- wait_timeout par défaut : 28800 secondes = 8 heures
SET PERSIST wait_timeout = 300;         -- 5 minutes
SET PERSIST interactive_timeout = 300;
```

## Tuer une requête ou une connexion

```sql
KILL QUERY 42;        -- annule la requête en cours, la connexion reste ouverte
KILL CONNECTION 42;   -- ferme la connexion entière
KILL 42;              -- synonyme court de KILL CONNECTION
```

## Cas particuliers

> [!info] `sys.processlist` ajoute des informations utiles à SHOW PROCESSLIST
> La vue `sys.processlist` (voir [[MySQL 16 — Performance Schema & sys schema]]) ajoute `trx_latency` (durée de la transaction) et `lock_latency` (temps passé en attente de verrou) — absents de `SHOW PROCESSLIST` brut.

## Pour aller plus loin

Les compteurs globaux et le rapport `SHOW ENGINE INNODB STATUS` sont détaillés dans [[MySQL 14 — SHOW GLOBAL STATUS & SHOW ENGINE INNODB STATUS]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
