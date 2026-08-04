#bdd #mariadb #fondamentaux

## Les bases système standard

Comme MySQL (voir [[MySQL 02 — Bases de données, schémas & bases système]]), MariaDB expose plusieurs bases internes au démarrage :

| Base | Rôle |
|------|------|
| `mysql` | Comptes utilisateurs, privilèges, tables de configuration interne |
| `information_schema` | Vues standard SQL exposant les métadonnées (tables, colonnes, index) |
| `performance_schema` | Instrumentation des performances (désactivée par défaut sur certaines installations légères) |
| `test` | Base vide créée par défaut avant `mariadb-secure-installation` (à supprimer) |

> [!info] `mysql` reste le nom de la base système
> Malgré le fork, MariaDB conserve le nom historique `mysql` pour sa base de métadonnées, par compatibilité avec les outils et scripts existants — il n'existe pas de base `mariadb` équivalente.

## mysql.global_priv : la différence structurelle depuis 10.4

Depuis MariaDB 10.4, la table `mysql.user` n'est plus une table de données mais une **vue** au-dessus d'une nouvelle table, `mysql.global_priv`, qui stocke les comptes et leurs privilèges globaux dans une seule colonne JSON plutôt qu'en colonnes séparées.

```sql
SELECT User, Host, JSON_VALUE(Priv, '$.plugin') AS plugin
FROM mysql.global_priv
LIMIT 5;
```

> [!warning] Ne pas éditer `mysql.global_priv` directement
> Comme pour `mysql.user` en MySQL, toute modification doit passer par `CREATE USER`/`ALTER USER`/`GRANT`/`REVOKE`. Le format JSON interne de `global_priv` est un détail d'implémentation qui peut changer entre versions mineures.

Toutes les tables système de MariaDB utilisent le moteur **Aria** (voir [[MariaDB 04 — Aria, le moteur des tables système]]) depuis la version 10.4, remplaçant l'ancien moteur MyISAM hérité de MySQL.

## Bases de données utilisateur

La création d'une base suit exactement la même syntaxe qu'en MySQL :

```sql
CREATE DATABASE IF NOT EXISTS lab_mariadb
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_uca1400_ai_ci;
```

> [!info] Une collation UCA plus récente que MySQL
> MariaDB propose depuis la version 10.10 des collations basées sur Unicode Collation Algorithm 14.0 (`utf8mb4_uca1400_ai_ci`), plus précises pour le tri international que les collations historiques `utf8mb4_general_ci`/`utf8mb4_unicode_ci` partagées avec MySQL. Ces dernières restent disponibles et compatibles.

## Pour aller plus loin

Les moteurs de stockage disponibles pour les tables utilisateur sont présentés dans [[MariaDB 03 — Moteurs de stockage, vue d'ensemble]].

Sources : [System-Versioned Tables — MariaDB Documentation](https://mariadb.com/docs/server/reference/sql-structure/temporal-tables/system-versioned-tables), [MDEV-20002 — Create system tables as aria instead of myisam](https://jira.mariadb.org/browse/MDEV-20002)
