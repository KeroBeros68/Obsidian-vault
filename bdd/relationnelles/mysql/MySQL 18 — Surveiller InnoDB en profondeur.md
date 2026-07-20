#bdd #mysql #innodb #monitoring #avancé

## Redo log : checkpoint et pression d'écriture

Le redo log (voir [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]]) garantit la durabilité — s'il se remplit plus vite que les checkpoints ne le libèrent, MySQL ralentit.

```sql
SHOW ENGINE INNODB STATUS\G
```

Section `LOG` :

```
Log sequence number          79456832
Log written up to            79456832
Log flushed up to            79456832
Last checkpoint at           79456832
```

> [!warning] LSN très en avance sur le dernier checkpoint = redo log sous pression
> Si `Log sequence number` s'éloigne significativement de `Last checkpoint at`, le redo log est sous pression et les écritures peuvent être ralenties.

```sql
SHOW VARIABLES LIKE 'innodb_redo_log_capacity';   -- 100 Mo par défaut en MySQL 8.4
SET PERSIST innodb_redo_log_capacity = 268435456; -- 256 Mo, pour une base à forte écriture
```

## Transactions : historique et purge lag

InnoDB maintient un historique de transactions pour le MVCC (lectures cohérentes sans verrou). Si l'undo log grossit, la purge est en retard.

```sql
SHOW ENGINE INNODB STATUS\G
```

Section `TRANSACTIONS` :

```
Trx id counter 12345
Purge done for trx's n:o < 12340 undo n:o < 0 state: running
History list length 23
```

| Métrique | Signification | Seuil d'alerte |
|----------|-------------------|---------------------|
| `History list length` | Transactions en attente de purge | > 10 000 |

> [!warning] Un History list length croissant signale un thread de purge dépassé
> Causes possibles : des `SELECT` très longs qui maintiennent une vue MVCC ancienne (empêchant la purge des versions antérieures), ou un nombre de `innodb_purge_threads` insuffisant pour le rythme des écritures.

## Adaptive hash index : utile ou pas ?

L'**adaptive hash index** (AHI) est un mécanisme automatique d'InnoDB qui construit un index hash en mémoire pour accélérer les lookups fréquents.

```sql
SHOW VARIABLES LIKE 'innodb_adaptive_hash_index';   -- OFF par défaut depuis MySQL 8.4 (activé en 8.0)
```

Pour évaluer son utilité réelle, consulter la section `INSERT BUFFER AND ADAPTIVE HASH INDEX` de `SHOW ENGINE INNODB STATUS` :

- `hash searches/s` élevé et `non-hash searches/s` faible → l'AHI est efficace, le garder.
- `hash searches/s` faible ou nul → l'AHI consomme de la mémoire pour rien, le désactiver.

```sql
SET PERSIST innodb_adaptive_hash_index = ON;    -- activer pour tester, dynamique
SET PERSIST innodb_adaptive_hash_index = OFF;   -- désactiver si aucun bénéfice observé
```

## Cas particuliers

> [!info] Ces métriques se lisent toutes dans le même rapport
> `SHOW ENGINE INNODB STATUS` reste la source unique pour le redo log, les transactions/purge et l'adaptive hash index — un seul rapport à consulter plutôt que plusieurs commandes séparées, une fois qu'on sait où regarder dans chaque section.

## Pour aller plus loin

Une fois un problème identifié (statistiques obsolètes, espace non récupéré), les actions de maintenance sont détaillées dans [[MySQL 19 — Maintenance des tables]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
