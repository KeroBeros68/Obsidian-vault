#bdd #mysql #sécurité #avancé

## Pourquoi la configuration par défaut ne suffit pas

Un MySQL avec les paramètres par défaut après `mysql_secure_installation` (voir [[MySQL 00 — Installation]]) est plus sûr qu'au premier démarrage, mais reste insuffisant pour la production : le compte root est un super-utilisateur omnipotent, aucun chiffrement TLS n'est forcé, les rôles ne sont pas séparés, et il n'y a pas d'audit des connexions. Chaque point est un vecteur d'attaque exploitable.

La démarche de durcissement suit une logique en couches : d'abord l'authentification, puis la gestion des comptes (voir [[MySQL 22 — Gestion des utilisateurs, verrouillage & validate_password]]), les rôles et privilèges (voir [[MySQL 23 — Rôles, privilèges granulaires & moindre privilège]]), le chiffrement du transport (voir [[MySQL 24 — Chiffrement TLS]]), et enfin le réseau et l'audit (voir [[MySQL 25 — Restreindre le réseau & auditer les connexions]]).

> [!info] Version utilisée
> Ce guide s'appuie sur MySQL 8.4 LTS sous Debian 12.

## `caching_sha2_password` : le plugin par défaut depuis MySQL 8.0

MySQL utilise un système de plugins d'authentification : chaque compte utilisateur est associé à un plugin qui gère la vérification du mot de passe. Depuis MySQL 8.0, le plugin par défaut est `caching_sha2_password`.

```sql
SELECT user, host, plugin
FROM mysql.user
WHERE user NOT IN ('mysql.sys','mysql.session','mysql.infoschema')
ORDER BY user, host;
```

Fonctionnement : le mot de passe est hashé en SHA-256 puis mis en cache côté serveur. La première authentification d'un utilisateur nécessite un échange sécurisé (connexion TLS, ou échange de clé RSA). Les authentifications suivantes utilisent le cache, d'où le nom « caching ».

| Plugin | Algorithme | Sécurité | Défaut depuis |
|--------|-----------|----------|---------------|
| `caching_sha2_password` | SHA-256 + cache | Forte | MySQL 8.0 |
| `mysql_native_password` | SHA-1 double | Faible (vulnérable au rejeu) | MySQL 4.1 à 5.7 |
| `sha256_password` | SHA-256 sans cache | Correcte mais lente | — |
| `auth_socket` | Identité Unix (comme `peer` dans PostgreSQL) | Forte | — |

> [!info] Différence avec PostgreSQL
> PostgreSQL utilise `scram-sha-256` dans `pg_hba.conf`, un standard (RFC 7677) résistant aux attaques par rejeu. MySQL a choisi son propre protocole `caching_sha2_password`, qui offre un niveau de sécurité similaire mais n'est pas compatible SCRAM. L'approche diffère aussi structurellement : chez PostgreSQL la méthode d'authentification est définie par ligne dans `pg_hba.conf` ; chez MySQL, le plugin est attaché au compte utilisateur.

## `mysql_native_password` : désactivé par défaut en 8.4

MySQL 8.4 a désactivé `mysql_native_password` par défaut — le plugin n'est plus chargé au démarrage :

```sql
SELECT PLUGIN_NAME, PLUGIN_STATUS
FROM information_schema.PLUGINS
WHERE PLUGIN_NAME = 'mysql_native_password';
-- PLUGIN_STATUS = DISABLED
```

Impact : les anciens clients ou connecteurs qui ne supportent pas `caching_sha2_password` ne peuvent plus se connecter. C'est le cas de PHP avec `mysqli` + `libmysqlclient` ancien (PHP < 7.4 sans `mysqlnd`), de Python avec `mysqlclient` < 2.0, et de certains outils legacy (anciens clients MySQL 5.x).

Si vous devez temporairement réactiver le plugin pour une migration :

```ini
[mysqld]
mysql_native_password = ON
```

> [!warning] Ne réactivez `mysql_native_password` que temporairement
> Ce plugin utilise un double hash SHA-1, vulnérable aux attaques par rejeu et aux rainbow tables. Déprécié depuis MySQL 8.0.34, désactivé par défaut en 8.4, et **supprimé en MySQL 9.0**. La réactivation doit rester temporaire : mettez à jour vos clients et migrez les comptes vers `caching_sha2_password` (voir [[MySQL 22 — Gestion des utilisateurs, verrouillage & validate_password]] pour `ALTER USER ... IDENTIFIED WITH`).

## `authentication_policy` : le contrôle multiniveau (MFA)

MySQL 8.4 supporte l'authentification multifacteur. Le paramètre `authentication_policy` définit les exigences pour chaque facteur :

```sql
SHOW VARIABLES LIKE 'authentication_policy';
-- Value = *,,
```

La valeur `*,,` signifie : premier facteur obligatoire (n'importe quel plugin), deuxième et troisième facteurs optionnels. Pour forcer `caching_sha2_password` comme premier facteur :

```sql
SET PERSIST authentication_policy = 'caching_sha2_password,,';
```

> [!warning] `default_authentication_plugin` supprimé en 8.4
> La variable `default_authentication_plugin`, utilisée dans les versions précédentes pour fixer le plugin par défaut, n'existe plus en MySQL 8.4. Le contrôle passe désormais entièrement par `authentication_policy`.

## Pour aller plus loin

La création de comptes restreints par host, le verrouillage et la politique de complexité des mots de passe sont détaillés dans [[MySQL 22 — Gestion des utilisateurs, verrouillage & validate_password]].

Sources : [Sécuriser MySQL : utilisateurs, TLS, rôles et moindre privilège — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/securisation/)
