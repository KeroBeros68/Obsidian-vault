#bdd #mysql #monitoring #intermédiaire

## L'équivalent MySQL de pg_stat_statements — mais sur disque

Le **slow query log** enregistre toutes les requêtes dont le temps d'exécution dépasse un seuil configurable. Contrairement à `pg_stat_statements` côté PostgreSQL (extension mémoire, statistiques agrégées en temps réel), MySQL écrit dans un **fichier de log** — l'équivalent en mémoire côté MySQL est `performance_schema.events_statements_summary_by_digest` (voir [[MySQL 16 — Performance Schema & sys schema]]).

## Activer le slow query log

```sql
SET PERSIST slow_query_log = ON;
SET PERSIST slow_query_log_file = '/var/lib/mysql/slow.log';
SET PERSIST long_query_time = 1;   -- seuil en secondes
```

`SET PERSIST` applique le changement immédiatement et le fait survivre à un redémarrage — sans modifier manuellement le fichier de configuration.

## Choisir le bon seuil (long_query_time)

| Valeur | Usage |
|--------|-------|
| `10` (défaut) | Trop élevé, ne capture que les requêtes catastrophiques |
| `1` | Bon point de départ pour la production |
| `0.5` | Pour un audit approfondi |
| `0` | Capture toutes les requêtes — attention à l'espace disque et aux I/O |

## Détecter les requêtes sans index, même rapides

```sql
SET PERSIST log_queries_not_using_indexes = ON;
SET PERSIST log_throttle_queries_not_using_indexes = 10;   -- limite le nombre de logs/minute
```

> [!tip] Une requête rapide aujourd'hui peut devenir lente demain
> Cette option capture les full table scans même quand ils sont encore rapides (petite table) — un excellent moyen de détecter des requêtes qui deviendront un problème quand la table grossira, avant que ça n'arrive.

## Analyser le log avec mysqldumpslow

```bash
mysqldumpslow -s t -t 10 /var/lib/mysql/slow.log    # top 10 par temps total
mysqldumpslow -s c -t 10 /var/lib/mysql/slow.log    # top 10 par nombre d'appels
mysqldumpslow -s at -t 10 /var/lib/mysql/slow.log   # top 10 par temps moyen
```

```
Count: 12  Time=2.34s (28s)  Lock=0.00s (0s)  Rows=90000.0 (1080000)
SELECT * FROM logs WHERE action = 'S'
```

Cette requête a été exécutée 12 fois, avec un temps moyen de 2,34 s et un temps total de 28 s — c'est la priorité d'optimisation, pas nécessairement la requête la plus lente unitairement.

## Cas particuliers

> [!warning] `long_query_time = 0` peut saturer le disque
> Capturer toutes les requêtes génère un volume de logs proportionnel au trafic total de l'instance — à réserver à une session d'audit ponctuelle et courte, jamais laissé actif en continu sur un serveur à fort trafic.

> [!info] Trier par temps total, pas par temps moyen, en premier
> Une requête à 5 ms exécutée 100 000 fois par jour pèse souvent plus sur le serveur qu'une requête à 2 s exécutée une seule fois — `mysqldumpslow -s t` (temps total) identifie mieux la priorité réelle que `-s at` (temps moyen) seul.

## Pour aller plus loin

L'équivalent en mémoire, sans passer par un fichier, est `sys.statement_analysis` — voir [[MySQL 16 — Performance Schema & sys schema]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
