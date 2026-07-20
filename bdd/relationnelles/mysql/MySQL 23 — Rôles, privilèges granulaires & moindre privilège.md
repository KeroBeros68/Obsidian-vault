#bdd #mysql #sécurité #avancé

## CREATE ROLE et GRANT rôle TO utilisateur

MySQL supporte les rôles depuis la version 8.0 : un rôle est un ensemble nommé de privilèges, attribuable à plusieurs utilisateurs.

```sql
-- Créer les rôles (pas de LOGIN — ce sont des groupes de privilèges)
CREATE ROLE 'role_reader', 'role_writer', 'role_admin';

-- Accorder des privilèges aux rôles
GRANT SELECT ON lab_mysql.* TO 'role_reader';
GRANT SELECT, INSERT, UPDATE, DELETE ON lab_mysql.* TO 'role_writer';
GRANT ALL PRIVILEGES ON lab_mysql.* TO 'role_admin';

-- Attribuer les rôles aux utilisateurs
GRANT 'role_reader' TO 'monitoring'@'10.0.1.50';
GRANT 'role_writer' TO 'app_writer'@'10.0.1.%';
GRANT 'role_admin' TO 'admin_app'@'localhost';
```

## SET DEFAULT ROLE : l'activation n'est pas automatique

Par défaut, les rôles accordés ne sont **pas** activés automatiquement à la connexion. L'utilisateur doit soit les activer manuellement (`SET ROLE`), soit disposer d'un rôle par défaut configuré :

```sql
-- Activer tous les rôles par défaut pour un utilisateur
SET DEFAULT ROLE ALL TO 'app_writer'@'10.0.1.%';
SET DEFAULT ROLE ALL TO 'monitoring'@'10.0.1.50';
SET DEFAULT ROLE ALL TO 'admin_app'@'localhost';
```

Ou activer l'activation automatique globale pour tous les utilisateurs :

```sql
SET PERSIST activate_all_roles_on_login = ON;
```

> [!warning] Piège classique : le rôle est accordé mais pas activé
> Si après un `GRANT 'role_reader' TO 'monitoring'@'10.0.1.50'`, l'utilisateur `monitoring` ne peut toujours pas lire les tables, c'est probablement que le rôle n'est pas activé. Vérifier avec `SELECT CURRENT_ROLE();` — si le résultat est `NONE`, exécuter `SET DEFAULT ROLE ALL TO user@host;`.

## Privilèges granulaires de MySQL 8.4

MySQL 8.4 a décomposé l'ancien privilège `SUPER` en privilèges plus fins :

| Privilège granulaire | Remplace (ancien `SUPER`) | Usage |
|----------------------|---------------------------|-------|
| `FLUSH_PRIVILEGES` | `FLUSH PRIVILEGES` | Recharger les tables de privilèges |
| `FLUSH_TABLES` | `FLUSH TABLES` | Vider les caches de tables |
| `OPTIMIZE_LOCAL_TABLE` | `OPTIMIZE TABLE` | Optimiser les tables (voir [[MySQL 19 — Maintenance des tables]]) |
| `SYSTEM_VARIABLES_ADMIN` | `SET GLOBAL`/`PERSIST` | Configuration runtime |
| `CONNECTION_ADMIN` | Gérer les connexions des autres | DBA uniquement |
| `ROLE_ADMIN` | `GRANT role TO user` | Gestion des rôles |

> [!info] `SET_ANY_DEFINER` : cas particulier
> `SET_ANY_DEFINER` n'est pas un remplacement direct de `SUPER` au même titre que les privilèges ci-dessus : il permet de créer des vues, triggers ou routines avec un `DEFINER` différent de l'utilisateur courant. À accorder uniquement aux comptes qui gèrent les migrations de schéma.

**Règle : n'accordez jamais `SUPER`, utilisez le privilège granulaire correspondant au besoin exact.**

## Moindre privilège : architecture recommandée

| Rôle | Privilèges | Compte type | Host |
|------|-----------|-------------|------|
| DBA | `ALL PRIVILEGES` + administration | `admin_app@localhost` | Socket local uniquement |
| Applicatif (écriture) | `SELECT, INSERT, UPDATE, DELETE` sur le schéma | `app_writer@10.0.1.%` | Réseau applicatif |
| Applicatif (lecture) | `SELECT` sur le schéma | `app_reader@10.0.1.%` | Réseau applicatif |
| Monitoring | `PROCESS, REPLICATION CLIENT, SELECT` sur `performance_schema` | `monitoring@10.0.1.50` | IP du serveur de monitoring |
| Backup | `SELECT, RELOAD, LOCK TABLES, SHOW VIEW, EVENT, TRIGGER` | `backup@localhost` | Local uniquement |

Créer le compte monitoring (lecture performance + processus, voir [[MySQL 20 — Monitoring Prometheus-Grafana & dépannage]]) :

```sql
CREATE USER 'monitoring'@'10.0.1.50'
  IDENTIFIED BY 'Monitor2026!Readonly';

GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'monitoring'@'10.0.1.50';
GRANT SELECT ON performance_schema.* TO 'monitoring'@'10.0.1.50';
GRANT SELECT ON sys.* TO 'monitoring'@'10.0.1.50';
```

Créer le compte backup (`mysqldump`, XtraBackup, voir [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]]) :

```sql
CREATE USER 'backup'@'localhost'
  IDENTIFIED BY 'Backup2026!Secure';

GRANT SELECT, RELOAD, LOCK TABLES, SHOW VIEW, EVENT, TRIGGER
  ON *.* TO 'backup'@'localhost';
GRANT BACKUP_ADMIN ON *.* TO 'backup'@'localhost';
```

Vérifier les privilèges effectifs d'un compte :

```sql
SHOW GRANTS FOR 'monitoring'@'10.0.1.50';
```

## Partial revokes : révoquer sur un schéma spécifique

MySQL 8.0.16+ supporte les *partial revokes* : la possibilité de révoquer un privilège global pour un schéma spécifique. Activer d'abord la fonctionnalité :

```sql
SET PERSIST partial_revokes = ON;
```

Exemple : donner `SELECT` global sauf sur le schéma `mysql` (qui contient les mots de passe) :

```sql
GRANT SELECT ON *.* TO 'app_reader'@'10.0.1.%';
REVOKE SELECT ON mysql.* FROM 'app_reader'@'10.0.1.%';
```

> [!warning] Effet de bord de `partial_revokes`
> Quand `partial_revokes = ON`, les caractères `%` et `_` non échappés dans les noms de schémas sont interprétés comme des caractères **littéraux** dans les affectations de privilèges partielles, pas comme des jokers. Par exemple, `GRANT SELECT ON `test_%`.* ...` ne cible que le schéma littéral `test_%`, pas tous les schémas commençant par `test_`.

## Éviter GRANT ALL et SUPER

| À éviter | Pourquoi | Alternative |
|----------|----------|--------------|
| `GRANT ALL PRIVILEGES ON *.*` | Donne tous les droits sur toutes les bases | Accorder par schéma + privilège exact |
| `GRANT SUPER` | Privilège monolithique en voie de dépréciation | Utiliser les privilèges granulaires 8.4 |
| `GRANT ALL ON db.*` | Inclut `DROP`, `ALTER`, `GRANT` | Lister `SELECT, INSERT, UPDATE, DELETE` |

## Pour aller plus loin

Une fois les comptes et privilèges en place, le chiffrement du transport reste indispensable — voir [[MySQL 24 — Chiffrement TLS]].

Sources : [Sécuriser MySQL : utilisateurs, TLS, rôles et moindre privilège — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/securisation/)
