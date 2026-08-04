#bdd #mariadb #avancé

## Fichiers de configuration

MariaDB lit sa configuration dans les mêmes emplacements que MySQL sur Debian/Ubuntu, avec un fichier dédié à sa propre arborescence :

```bash
/etc/mysql/mariadb.conf.d/50-server.cnf   # Configuration principale du serveur
/etc/mysql/mariadb.conf.d/50-client.cnf   # Configuration du client par défaut
/etc/mysql/conf.d/                        # Fragments additionnels (*.cnf)
```

```sql
SHOW VARIABLES LIKE 'version_compile_os';
```

> [!info] `[mysqld]` reste le nom de section
> Par compatibilité historique, la section de configuration du serveur MariaDB s'appelle toujours `[mysqld]` dans les fichiers `.cnf` — pas `[mariadbd]`. C'est un des rares endroits où l'ancien nom persiste explicitement dans la configuration.

## Paramètres InnoDB : mêmes leviers que MySQL

```ini
[mysqld]
innodb_buffer_pool_size = 4G
innodb_flush_log_at_trx_commit = 1
innodb_file_per_table = ON
max_connections = 200
```

Le dimensionnement suit les mêmes règles que pour MySQL (voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]) : `innodb_buffer_pool_size` autour de 50-80 % de la RAM disponible sur un serveur dédié à MariaDB.

## Paramètres Aria : propres à MariaDB

```ini
[mysqld]
aria_pagecache_buffer_size = 256M
aria_block_size = 8192
aria_log_file_size = 1G
```

Le cache de pages Aria (voir [[MariaDB 04 — Aria, le moteur des tables système]]) mérite une attention particulière même sur une instance qui n'utilise Aria que pour ses tables système : un cache trop petit ralentit les opérations sur `information_schema` et les métadonnées de comptes.

## Recharger la configuration

```sql
-- Modification à chaud, sans redémarrage (variables dynamiques) — non persistant
SET GLOBAL innodb_buffer_pool_size = 4294967296;
```

```bash
sudo systemctl restart mariadb
```

> [!warning] MariaDB n'a pas d'équivalent de SET PERSIST
> MySQL 8.0+ écrit les changements `SET PERSIST` dans `mysqld-auto.cnf`, appliqués automatiquement au redémarrage suivant (voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]). MariaDB ne propose aucun mécanisme équivalent : un `SET GLOBAL` ne survit jamais à un redémarrage — le changement doit être reporté manuellement dans le fichier `.cnf` pour devenir permanent.

## Vérifier une variable et sa source

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SELECT @@GLOBAL.innodb_buffer_pool_size;
```

## Pour aller plus loin

La sauvegarde et la restauration, avec les deux outils dédiés `mariadb-dump` et `mariadb-backup`, sont détaillées dans [[MariaDB 10 — Sauvegarde et restauration (mariadb-dump & mariadb-backup)]].

Sources : [MariaDB Package Repository Setup and Usage — MariaDB Documentation](https://mariadb.com/docs/server/server-management/install-and-upgrade-mariadb/installing-mariadb/binary-packages/mariadb-package-repository-setup-and-usage), [Aria Storage Engine — MariaDB Documentation](https://mariadb.com/docs/server/server-usage/storage-engines/aria/aria-storage-engine)
