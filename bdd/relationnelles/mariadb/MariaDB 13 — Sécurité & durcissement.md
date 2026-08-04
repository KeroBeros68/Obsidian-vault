#bdd #mariadb #sécurité #avancé

## Les mêmes principes que MySQL, avec des mécanismes propres

Le durcissement d'une instance MariaDB suit la même logique en couches que MySQL (voir [[MySQL 21 — Authentification (caching_sha2_password & authentication_policy)]] et les fiches suivantes) : authentification, comptes restreints, privilèges minimaux, chiffrement du transport, restriction réseau. Les commandes SQL sont très proches, mais quelques mécanismes diffèrent.

## Comptes restreints par host

```sql
CREATE USER 'app_writer'@'10.0.1.%' IDENTIFIED VIA ed25519 USING PASSWORD('AppWriter2026!');
GRANT SELECT, INSERT, UPDATE, DELETE ON lab_mariadb.* TO 'app_writer'@'10.0.1.%';
```

Comme pour MySQL, `user@host` forme un couple unique, et `@'%'` reste à éviter pour tout compte disposant de privilèges d'écriture (voir [[MySQL 22 — Gestion des utilisateurs, verrouillage & validate_password]] pour la logique complète, transposable telle quelle).

## Rôles

MariaDB a introduit les rôles avant MySQL (dès la version 10.0.5) :

```sql
CREATE ROLE role_lecture;
GRANT SELECT ON lab_mariadb.* TO role_lecture;
GRANT role_lecture TO 'app_reader'@'10.0.1.%';

SET DEFAULT ROLE role_lecture FOR 'app_reader'@'10.0.1.%';
```

> [!warning] Un rôle accordé n'est pas actif par défaut
> Comme sur MySQL (voir [[MySQL 23 — Rôles, privilèges granulaires & moindre privilège]]), un rôle accordé via `GRANT` doit être activé — soit avec `SET DEFAULT ROLE`, soit manuellement en session avec `SET ROLE role_lecture;`. Vérifier avec `SELECT CURRENT_ROLE();`.

## TLS

```ini
[mysqld]
ssl_ca = /etc/mysql/ssl/ca.pem
ssl_cert = /etc/mysql/ssl/server-cert.pem
ssl_key = /etc/mysql/ssl/server-key.pem
```

```sql
CREATE USER 'app_secure'@'10.0.1.%' REQUIRE SSL;
```

Les mêmes clauses `REQUIRE SSL`/`REQUIRE X509` que MySQL sont supportées (voir [[MySQL 24 — Chiffrement TLS]]) — la syntaxe est partagée entre les deux moteurs.

## Restriction réseau

```ini
[mysqld]
bind_address = 127.0.0.1,10.0.1.5
skip_name_resolve = ON
```

Mêmes paramètres, mêmes recommandations que MySQL (voir [[MySQL 25 — Restreindre le réseau & auditer les connexions]]) : ne jamais écouter sur `0.0.0.0` sans pare-feu, désactiver la résolution DNS inversée pour accélérer les connexions et réduire la dépendance au DNS.

## Checklist de durcissement

| Vérification | Commande | Attendu |
|----------------|----------|---------|
| root protégé par `unix_socket` ou mot de passe fort | `SELECT User, Host, plugin FROM mysql.global_priv WHERE User='root';` | `unix_socket` en local, jamais `@'%'` |
| `ed25519` installé pour les comptes réseau | `SHOW PLUGINS LIKE 'ed25519';` | `ACTIVE` |
| Pas de compte `@'%'` avec privilèges élevés | `SELECT User, Host FROM mysql.global_priv WHERE Host='%';` | Aucune ligne à privilèges élevés |
| TLS configuré | `SHOW VARIABLES LIKE 'ssl_cert';` | Chemin non vide |
| `bind_address` restreint | `SHOW VARIABLES LIKE 'bind_address';` | Pas `0.0.0.0` |
| Base `test` supprimée | `SHOW DATABASES LIKE 'test';` | Aucune ligne |

## Pour aller plus loin

Cela conclut le module MariaDB — voir [[MariaDB — Index des fiches]] pour une vue d'ensemble, ou [[BDD — Home]] pour explorer les autres moteurs (MySQL, SQL).

Sources : [Authentication Plugin - Unix Socket — MariaDB Documentation](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-unix-socket), [Incompatibilities and Feature Differences Between MariaDB and MySQL — MariaDB Documentation](https://mariadb.com/docs/release-notes/community-server/about/compatibility-and-differences/incompatibilities-and-feature-differences-between-mariadb-and-mysql-unmaint/incompatibilities-and-feature-differences-between-mariadb-10-4-and-mysql-8)
