#bdd #mariadb #avancé

## Le même moteur, deux évolutions séparées depuis 2009

InnoDB reste le moteur de stockage par défaut de MariaDB pour les tables utilisateur, exactement comme sur MySQL. Les concepts fondamentaux — buffer pool, redo log, doublewrite buffer, tablespaces, MVCC, verrouillage au niveau ligne — sont partagés et déjà détaillés dans le module MySQL (voir [[MySQL 05 — InnoDB — le buffer pool]] et [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]]) : ces mécanismes s'appliquent sans changement conceptuel à MariaDB.

> [!info] Un fork du fork, historiquement
> Le code InnoDB de MariaDB a divergé de celui d'Oracle au fil des versions : MariaDB applique ses propres correctifs et optimisations à sa copie d'InnoDB, sans toujours réintégrer les évolutions apportées par Oracle côté MySQL (et inversement). Les deux restent compatibles au niveau du format de fichier `.ibd` dans la plupart des cas, mais pas garantis identiques en interne.

## Réglages communs, valeurs par défaut parfois différentes

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
SHOW VARIABLES LIKE 'innodb_file_per_table';
```

Les mêmes principes de dimensionnement s'appliquent (voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]) : `innodb_buffer_pool_size` à 50-80 % de la RAM disponible sur un serveur dédié, `innodb_flush_log_at_trx_commit = 1` pour la durabilité maximale.

> [!warning] Ne pas copier une configuration MySQL sans vérifier les noms de variables
> Certaines variables InnoDB portent des noms différents ou n'existent que d'un seul côté. Par exemple, `innodb_dedicated_server` (auto-dimensionnement du buffer pool, voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]) est une fonctionnalité MySQL qui n'a pas d'équivalent direct dans MariaDB — le dimensionnement y reste manuel.

## Chiffrement InnoDB : une implémentation différente

MariaDB et MySQL chiffrent tous deux les tablespaces InnoDB au repos, mais avec des implémentations distinctes (plugins de gestion de clés différents, granularité de configuration différente). Un tablespace chiffré par l'un ne peut pas être ouvert tel quel par l'autre.

## Pour aller plus loin

Les deux fonctionnalités qui distinguent le plus MariaDB de MySQL au niveau du langage SQL — Sequences et tables System-Versioned — sont couvertes dans [[MariaDB 06 — Sequences & tables System-Versioned]].

Sources : [Storage Engines | MariaDB Platform — MariaDB Documentation](https://mariadb.com/docs/platform/mariadb-faqs/storage-engines), [Incompatibilities and Feature Differences Between MariaDB and MySQL — MariaDB Documentation](https://mariadb.com/docs/release-notes/community-server/about/compatibility-and-differences/incompatibilities-and-feature-differences-between-mariadb-and-mysql-unmaint/incompatibilities-and-feature-differences-between-mariadb-10-4-and-mysql-8)
