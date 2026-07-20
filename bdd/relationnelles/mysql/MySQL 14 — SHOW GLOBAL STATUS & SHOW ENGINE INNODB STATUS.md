#bdd #mysql #monitoring #intermédiaire

## SHOW GLOBAL STATUS : les compteurs cumulatifs clés

`SHOW GLOBAL STATUS` expose des centaines de compteurs depuis le démarrage du serveur. Les plus utiles :

```sql
SHOW GLOBAL STATUS WHERE Variable_name IN (
  'Threads_connected', 'Threads_running', 'Max_used_connections', 'Connections',
  'Slow_queries', 'Questions',
  'Innodb_buffer_pool_read_requests', 'Innodb_buffer_pool_reads',
  'Innodb_row_lock_waits', 'Innodb_row_lock_time'
);
```

| Compteur | Signification |
|----------|-------------------|
| `Threads_connected` | Connexions actuellement ouvertes |
| `Threads_running` | Connexions exécutant une requête en ce moment |
| `Max_used_connections` | Pic de connexions simultanées depuis le démarrage |
| `Slow_queries` | Nombre total de requêtes lentes détectées |
| `Innodb_buffer_pool_read_requests` | Lectures logiques (depuis le buffer pool) |
| `Innodb_buffer_pool_reads` | Lectures physiques (depuis le disque) |
| `Innodb_row_lock_waits` | Attentes de verrou de ligne |

## Calculer le buffer pool hit ratio

```sql
SELECT (1 - (
  (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads')
  /
  (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests')
)) * 100 AS buffer_pool_hit_ratio;
```

> [!tip] Seuil de référence
> Un ratio supérieur à 99% est normal pour une charge OLTP. En dessous de 95%, augmenter `innodb_buffer_pool_size` (voir [[MySQL 05 — InnoDB — le buffer pool]]).

## SHOW ENGINE INNODB STATUS : le diagnostic instantané le plus riche

```sql
SHOW ENGINE INNODB STATUS\G
```

| Section | Ce qu'elle montre |
|---------|------------------------|
| `SEMAPHORES` | Attentes de verrous internes (mutex, rw-lock) |
| `TRANSACTIONS` | Transactions actives, historique, purge — voir [[MySQL 18 — Surveiller InnoDB en profondeur]] |
| `FILE I/O` | Threads I/O, opérations en attente |
| `BUFFER POOL AND MEMORY` | Taille du pool, pages free/dirty/data, hit rate |
| `ROW OPERATIONS` | INSERT/UPDATE/DELETE par seconde |
| `LOG` | Position du redo log, séquence LSN |
| `DEADLOCKS` | Dernier deadlock détecté par InnoDB |

> [!warning] Seul le dernier deadlock est affiché
> `SHOW ENGINE INNODB STATUS` ne montre que le deadlock le plus récent. Pour enregistrer **tous** les deadlocks dans le error log — indispensable pour analyser des deadlocks récurrents en production :
> ```sql
> SET PERSIST innodb_print_all_deadlocks = ON;
> ```

## Cas particuliers

> [!info] Compteurs cumulatifs vs instantanés
> `SHOW GLOBAL STATUS` accumule depuis le démarrage du serveur : un `Slow_queries` élevé peut simplement refléter une longue durée de fonctionnement. Comparer deux relevés à intervalle régulier (delta) donne une image plus juste de l'activité récente qu'une lecture unique.

## Pour aller plus loin

Le slow query log, pour identifier précisément quelles requêtes posent problème, est couvert dans [[MySQL 15 — Le slow query log]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
