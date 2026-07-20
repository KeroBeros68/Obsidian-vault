#bdd #mysql #réplication #avancé

## Lab utilisé

| Rôle | Hostname | IP | OS | MySQL |
|------|----------|-----|-----|-------|
| Source | mysql-source | 192.168.122.71 | Debian 12 | 8.4 LTS |
| Replica | mysql-replica | 192.168.122.72 | Debian 12 | 8.4 LTS |

Prérequis : deux serveurs MySQL 8.4 installés (voir [[MySQL 00 — Installation]]), un réseau autorisant le TCP entre les deux nœuds (port 3306), un accès root sur les deux, et les bases de la configuration binlog (voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]).

## Configurer le source

Éditer `/etc/mysql/mysql.conf.d/mysqld.cnf` sur le source :

```ini
[mysqld]
# Identifiant unique du serveur — DOIT être différent sur chaque nœud
server_id = 1

# Binary log (activé par défaut en 8.4, mais nommage explicite)
log_bin = /var/lib/mysql/binlog

# GTID obligatoire
gtid_mode = ON
enforce_gtid_consistency = ON

# Format ROW pour la cohérence (défaut en 8.4)
binlog_format = ROW

# Durabilité maximale
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1

# Rétention binlog (30 jours)
binlog_expire_logs_seconds = 2592000
```

```bash
sudo systemctl restart mysql
```

```sql
SHOW VARIABLES LIKE 'gtid_mode';               -- ON
SHOW VARIABLES LIKE 'enforce_gtid_consistency'; -- ON
SHOW VARIABLES LIKE 'server_id';                -- 1
```

> [!warning] `server_id` unique et non nul
> Chaque serveur du cluster de réplication doit avoir un `server_id` unique et supérieur à 0. Si deux serveurs partagent le même ID, la réplication ne fonctionne pas. Convention : `1` pour le source, `2` pour le premier replica, `3` pour le deuxième, etc.

### Créer un rôle REPLICATION SLAVE dédié

Ne pas utiliser le compte root pour la réplication — créer un rôle dédié avec les seuls privilèges nécessaires (voir [[MySQL 23 — Rôles, privilèges granulaires & moindre privilège]]) :

```sql
CREATE USER 'replicator'@'192.168.122.72' IDENTIFIED BY 'motdepasse_fort_replication';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'192.168.122.72';
```

Le privilège `REPLICATION SLAVE` ne permet que la lecture du binary log, pas d'accès aux données.

> [!warning] Identifiants stockés en clair sur le replica
> Les identifiants du compte de réplication sont stockés en clair dans le repository de métadonnées de réplication côté replica (`mysql.slave_master_info`). Créer un compte dédié avec le strict minimum de privilèges et protéger la connexion avec TLS en production (voir [[MySQL 24 — Chiffrement TLS]]).

> [!info] Différence avec PostgreSQL
> En PostgreSQL, le rôle de réplication a l'attribut `REPLICATION` et on autorise la connexion via une ligne `replication` dans `pg_hba.conf`. En MySQL, le privilège `REPLICATION SLAVE` joue le même rôle, et l'autorisation réseau passe par le `@host` dans `CREATE USER`.

### Autoriser la connexion du replica

Si `bind_address` est à `127.0.0.1`, le replica ne peut pas se connecter (voir [[MySQL 25 — Restreindre le réseau & auditer les connexions]]) :

```ini
[mysqld]
bind_address = 0.0.0.0
```

```bash
ss -tlnp | grep 3306
# LISTEN 0 151 0.0.0.0:3306 0.0.0.0:* users:(("mysqld",pid=1234,fd=33))
```

## Initialiser le replica

Éditer `/etc/mysql/mysql.conf.d/mysqld.cnf` sur le replica :

```ini
[mysqld]
# Identifiant unique — différent du source
server_id = 2

# GTID obligatoire
gtid_mode = ON
enforce_gtid_consistency = ON

# Le replica en lecture seule
read_only = ON
super_read_only = ON

# Binary log activé (utile pour la chaîne de réplication ou le failover)
log_bin = /var/lib/mysql/binlog

# Relay log
relay_log = /var/lib/mysql/relay-bin

# Applier multi-thread (accélère le rattrapage)
replica_parallel_workers = 4
replica_parallel_type = LOGICAL_CLOCK
replica_preserve_commit_order = ON
```

```bash
sudo systemctl restart mysql
```

### Clone plugin : copie à chaud simplifiée

Le clone plugin (intégré à MySQL 8.0+) copie l'intégralité du data directory depuis le source vers le replica — l'équivalent MySQL de `pg_basebackup -R` pour PostgreSQL.

```sql
-- Installer le plugin sur le source ET le replica
INSTALL PLUGIN clone SONAME 'mysql_clone.so';

-- Sur le source (donor) : créer un utilisateur clone
CREATE USER 'clone_user'@'192.168.122.72' IDENTIFIED BY 'motdepasse_clone';
GRANT BACKUP_ADMIN ON *.* TO 'clone_user'@'192.168.122.72';

-- Sur le replica (recipient) : accorder CLONE_ADMIN
GRANT CLONE_ADMIN ON *.* TO 'root'@'localhost';

-- Sur le replica : configurer le donor autorisé
SET GLOBAL clone_valid_donor_list = '192.168.122.71:3306';
```

> [!warning] Le clonage efface les données du recipient
> Le clonage distant supprime les données utilisateur et les binary logs du recipient avant la copie. Le serveur recipient redémarre automatiquement à la fin de l'opération. N'exécuter cette commande que sur un serveur que vous êtes prêt à réinitialiser.

```sql
-- Depuis le replica
CLONE INSTANCE FROM 'clone_user'@'192.168.122.71':3306
  IDENTIFIED BY 'motdepasse_clone';
```

Le replica redémarre automatiquement avec les données du source.

> [!info] Alternative : mysqldump ou MySQL Shell
> Si le clone plugin n'est pas disponible :
> ```bash
> mysqldump -u root -p --all-databases --single-transaction \
>   --routines --triggers --events --set-gtid-purged=ON \
>   -h 192.168.122.71 > /tmp/source_dump.sql
> mysql -u root -p < /tmp/source_dump.sql
> ```
> Ou avec MySQL Shell `util.dumpInstance()` / `util.loadDump()` pour un transfert parallèle — voir [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]].

## CHANGE REPLICATION SOURCE TO avec GTID

Une fois les données synchronisées (clone, dump ou snapshot), configurer la réplication sur le replica :

```sql
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST = '192.168.122.71',
  SOURCE_PORT = 3306,
  SOURCE_USER = 'replicator',
  SOURCE_PASSWORD = 'motdepasse_fort_replication',
  SOURCE_AUTO_POSITION = 1,
  GET_SOURCE_PUBLIC_KEY = 1;
```

| Option | Rôle |
|--------|------|
| `SOURCE_HOST` | Adresse du source |
| `SOURCE_USER` | Rôle de réplication |
| `SOURCE_AUTO_POSITION = 1` | Active le repositionnement automatique par GTID — pas besoin de spécifier le fichier binlog et l'offset |
| `GET_SOURCE_PUBLIC_KEY = 1` | Récupère la clé publique du source pour `caching_sha2_password` |

> [!warning] GET_SOURCE_PUBLIC_KEY vs TLS
> `GET_SOURCE_PUBLIC_KEY = 1` récupère la clé publique à la volée depuis le source, ce qui expose à une attaque *man-in-the-middle* sur la première connexion. En production, préférer une connexion TLS (`SOURCE_SSL = 1`) ou fournir la clé publique localement avec `SOURCE_PUBLIC_KEY_PATH`.

> [!info] Terminologie MySQL 8.4
> Les commandes `CHANGE MASTER TO`, `SHOW SLAVE STATUS`, `START SLAVE` sont dépréciées depuis MySQL 8.0.22 et remplacées par `CHANGE REPLICATION SOURCE TO`, `SHOW REPLICA STATUS`, `START REPLICA`. Les anciennes commandes fonctionnent encore mais émettent des warnings.

```sql
START REPLICA;
SHOW REPLICA STATUS\G
```

```
Replica_IO_Running: Yes
Replica_SQL_Running: Yes
Source_Log_File: binlog.000003
Seconds_Behind_Source: 0
Retrieved_Gtid_Set: 3e11fa47-71ca-11e1-9e33-c80aa9429562:1-23
Executed_Gtid_Set: 3e11fa47-71ca-11e1-9e33-c80aa9429562:1-23
```

Les trois indicateurs critiques : `Replica_IO_Running: Yes` (le thread I/O reçoit les binlogs du source), `Replica_SQL_Running: Yes` (le thread SQL applique les événements), `Seconds_Behind_Source: 0` (le replica est synchronisé).

## Pour aller plus loin

La surveillance détaillée du lag et le tuning de l'applier multi-thread sont couverts dans [[MySQL 28 — Surveiller la réplication & calculer le lag]].

Sources : [Réplication MySQL : source-replica, GTID et haute disponibilité — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/replication/)
