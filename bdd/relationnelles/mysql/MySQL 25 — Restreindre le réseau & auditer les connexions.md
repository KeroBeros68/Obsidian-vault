#bdd #mysql #sécurité #avancé

## bind_address et skip_name_resolve : rappel

La restriction de `bind_address` (ne pas écouter sur `0.0.0.0`) et la désactivation du DNS inversé via `skip_name_resolve` sont des réglages de `mysqld.cnf` déjà détaillés dans [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]. En synthèse pour le durcissement :

| Configuration | Exposition | Usage |
|----------------|------------|-------|
| `bind_address = 127.0.0.1` | Local uniquement | Dev, lab |
| `bind_address = 127.0.0.1,10.0.1.5` | Local + une IP réseau | Production, réseau applicatif dédié |
| `bind_address = 0.0.0.0` | Toutes les interfaces | Déconseillé sauf derrière un firewall/proxy |

> [!info] Après activation de `skip_name_resolve`
> N'utiliser que des adresses IP (jamais de noms d'hôte) dans la partie `@host` des comptes `CREATE USER` — voir [[MySQL 22 — Gestion des utilisateurs, verrouillage & validate_password]].

## Firewall système : une couche supplémentaire

`bind_address` restreint l'écoute côté MySQL, mais un firewall système ajoute une couche de défense indépendante du service :

```bash
# nftables (Debian 12+)
nft add rule inet filter input tcp dport 3306 ip saddr 10.0.1.0/24 accept
nft add rule inet filter input tcp dport 3306 drop

# ufw (Ubuntu)
sudo ufw allow from 10.0.1.0/24 to any port 3306
sudo ufw deny 3306

# firewalld (RHEL/Rocky)
sudo firewall-cmd --new-zone=mysql --permanent
sudo firewall-cmd --zone=mysql --add-source=10.0.1.0/24 --permanent
sudo firewall-cmd --zone=mysql --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```

> [!tip] Trois couches de restriction réseau
> `bind_address` restreint + `skip_name_resolve` activé + firewall système : chaque couche protège contre une classe d'erreur différente (mauvaise configuration MySQL, résolution DNS compromise, règles réseau contournées).

## connection_control : limiter le brute-force

Le plugin `connection_control` ralentit les connexions après un certain nombre d'échecs — une protection contre les attaques par force brute :

```sql
-- Installer le plugin
INSTALL PLUGIN CONNECTION_CONTROL SONAME 'connection_control.so';
INSTALL PLUGIN CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS
  SONAME 'connection_control.so';

-- Configurer
SET PERSIST connection_control_failed_connections_threshold = 3;
SET PERSIST connection_control_min_connection_delay = 1000;      -- 1 seconde
SET PERSIST connection_control_max_connection_delay = 86400000;  -- 24 heures max
```

Après 3 échecs d'authentification, MySQL impose un délai croissant avant chaque nouvelle tentative.

```sql
SELECT * FROM information_schema.CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS;
```

## Auditer avec le slow query log

Le slow query log, configuré dans [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]] et exploité dans [[MySQL 15 — Le slow query log]], sert aussi d'outil d'audit basique : il trace les requêtes qui dépassent un seuil de temps et les requêtes sans index.

## Audit plugin (MySQL Enterprise) vs alternatives open source

| Solution | Licence | Fonctionnalités |
|----------|---------|-------------------|
| MySQL Enterprise Audit | Commercial | Audit complet (connexions, requêtes, DDL), format XML/JSON, filtrage par user/event |
| Percona Audit Log | Open source (Percona Server) | Compatible Enterprise Audit, format XML/JSON |
| MariaDB Audit Plugin | Open source | Compatible MySQL Community (avec limitations de version) |
| McAfee MySQL Audit Plugin | Open source (archivé) | Audit basique, plus maintenu |

Pour MySQL Community, la combinaison `slow_query_log` + `general_log` (temporaire) + `connection_control` + `log_error` couvre les besoins basiques d'audit. Pour un audit complet, envisager Percona Server ou MySQL Enterprise.

## Checklist de durcissement

| Vérification | Commande | Attendu |
|----------------|----------|---------|
| Pas de compte sans mot de passe | `SELECT user, host FROM mysql.user WHERE authentication_string = '';` | Aucune ligne |
| Pas de `@'%'` pour l'admin | `SELECT user, host FROM mysql.user WHERE host = '%' AND Super_priv = 'Y';` | Aucune ligne |
| TLS configuré | `SHOW VARIABLES LIKE 'ssl_cert';` | Chemin vers le certificat (non vide) |
| Connexion courante chiffrée | `SHOW SESSION STATUS LIKE 'Ssl_cipher';` | Non vide |
| `validate_password` actif | `SHOW VARIABLES LIKE 'validate_password.policy';` | `MEDIUM` ou `STRONG` |
| `mysql_native_password` désactivé | `SELECT PLUGIN_STATUS FROM information_schema.PLUGINS WHERE PLUGIN_NAME='mysql_native_password';` | `DISABLED` |
| `bind_address` restreint | `SHOW VARIABLES LIKE 'bind_address';` | Pas `0.0.0.0` ni `*` |
| `skip_name_resolve` activé | `SHOW VARIABLES LIKE 'skip_name_resolve';` | `ON` |
| Rôles séparés | `SELECT user, host FROM mysql.user;` | Admin, writer, reader, monitoring, backup distincts |
| `connection_control` installé | `SHOW PLUGINS LIKE 'CONNECTION_CONTROL%';` | `ACTIVE` |
| Slow query log activé | `SHOW VARIABLES LIKE 'slow_query_log';` | `ON` |

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| `Authentication plugin 'caching_sha2_password' cannot be loaded` | Client MySQL ancien (5.x) sans support SHA2 | Mettre à jour le client ou réactiver temporairement `mysql_native_password` |
| `Access denied` après migration vers 8.4 | `mysql_native_password` désactivé | `ALTER USER ... IDENTIFIED WITH caching_sha2_password BY '...'` |
| `ERROR 2026: SSL connection error` | Certificat serveur expiré ou invalide | Vérifier `Ssl_server_not_after`, renouveler puis `ALTER INSTANCE RELOAD TLS` |
| Connexion TLS impossible depuis l'application | L'application ne passe pas `--ssl-mode=REQUIRED` | Configurer le connecteur avec SSL obligatoire |
| `SELECT CURRENT_ROLE()` retourne `NONE` | Les rôles ne sont pas activés par défaut | `SET DEFAULT ROLE ALL TO user@host;` ou `SET PERSIST activate_all_roles_on_login = ON;` |
| Rôle accordé mais privilèges absents | `activate_all_roles_on_login = OFF` | `SET PERSIST activate_all_roles_on_login = ON;` |
| Oubli du mot de passe root | — | Redémarrer avec `--skip-grant-tables`, réinitialiser puis redémarrer normalement (voir [[MySQL 00 — Installation]]) |
| `validate_password` bloque la création | Mot de passe trop simple | Respecter la politique (longueur, majuscule, chiffre, spécial) ou baisser temporairement la policy |

## À retenir

- `caching_sha2_password` est le plugin par défaut (contrôlé par `authentication_policy`, plus par `default_authentication_plugin` qui n'existe plus en 8.4). `mysql_native_password` est déprécié depuis 8.0.34, désactivé par défaut en 8.4, et supprimé en 9.0.
- La restriction réseau se fait au niveau du compte (`@host`), contrairement à PostgreSQL qui utilise `pg_hba.conf`. Un compte `@'%'` avec des privilèges élevés est un risque majeur.
- `REQUIRE SSL` dans `CREATE USER` force le chiffrement ; ajouter `--ssl-mode=VERIFY_CA` ou `VERIFY_IDENTITY` côté client pour vérifier l'identité du serveur.
- Les rôles ne sont pas activés automatiquement à la connexion — penser à `SET DEFAULT ROLE ALL` ou `activate_all_roles_on_login = ON`.
- N'accorder jamais `GRANT ALL ON *.*` ni `SUPER` — utiliser les privilèges granulaires de MySQL 8.4 (`FLUSH_PRIVILEGES`, `SYSTEM_VARIABLES_ADMIN`, etc.).
- `partial_revokes` permet de révoquer un privilège global sur un schéma précis, indispensable pour protéger le schéma `mysql`.
- Installer `connection_control` pour ralentir le brute-force après 3 tentatives échouées.
- `bind_address = 127.0.0.1` + `skip_name_resolve = ON` + firewall système : trois couches de restriction réseau.
- Le slow query log (`long_query_time = 1`) sert aussi d'outil d'audit — l'activer dès le premier jour.

## Pour aller plus loin

La sécurisation complète du transport (TLS) est détaillée dans [[MySQL 24 — Chiffrement TLS]]. La réplication (GTID, source-réplica, Group Replication, InnoDB Cluster) est couverte à partir de [[MySQL 26 — Concepts de réplication & GTID]].

Sources : [Sécuriser MySQL : utilisateurs, TLS, rôles et moindre privilège — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/securisation/)
