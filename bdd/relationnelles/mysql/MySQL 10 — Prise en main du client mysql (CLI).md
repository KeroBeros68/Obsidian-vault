#bdd #mysql #fondamentaux #cli

## Se connecter à MySQL

### Connexion locale (socket Unix)

La méthode la plus courante sur le serveur lui-même. La connexion passe par un socket Unix, pas de réseau, pas de chiffrement nécessaire :

```bash
mysql -u root -p
```

```
Enter password:
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.4.5 MySQL Community Server - GPL
mysql>
```

Pour se connecter directement à une base spécifique :

```bash
mysql -u root -p labdb
```

### Connexion réseau (TCP/IP)

```bash
mysql -h 192.168.122.70 -P 3306 -u labadmin -p labdb
```

| Option | Description |
|--------|-------------|
| `-h` | Adresse du serveur (hostname ou IP) |
| `-P` | Port (3306 par défaut) |
| `-u` | Nom d'utilisateur |
| `-p` | Demander le mot de passe (sans espace après `-p`) |
| `labdb` | Base de données à sélectionner au démarrage |

> [!warning] `-p` seul, jamais `-pMOTDEPASSE` collé
> `-p` seul demande le mot de passe en mode interactif (sécurisé). `-pmotdepasse` (collé, sans espace) le passe en argument shell : visible dans `ps aux` et dans l'historique. À ne jamais faire en production.

### Fichier d'options `~/.my.cnf`

Plutôt que de retaper les options de connexion à chaque fois :

```ini
[client]
user=labadmin
password=MotDePasseFort!2026
host=127.0.0.1
port=3306
```

```bash
chmod 600 ~/.my.cnf
```

> [!warning] `chmod 600` indispensable, pas optionnel
> MySQL ignore silencieusement les fichiers de configuration world-writable. Le mot de passe y est stocké en clair — acceptable pour un lab, à éviter en production (voir `mysql_config_editor` ci-dessous).

Ensuite, une simple commande suffit :

```bash
mysql labdb
```

### `mysql_config_editor` : identifiants chiffrés (login-path)

```bash
mysql_config_editor set --login-path=lab --host=127.0.0.1 --user=labadmin --password
```

L'outil demande le mot de passe et le stocke chiffré dans `~/.mylogin.cnf`. Pour l'utiliser :

```bash
mysql --login-path=lab labdb
```

Pour lister les profils enregistrés (mot de passe masqué, jamais affiché en clair) :

```bash
mysql_config_editor print --all
```

> [!tip] Préférer `login-path` à `~/.my.cnf`
> `~/.mylogin.cnf` n'expose jamais le mot de passe en clair sur la ligne de commande ni dans un fichier lisible, et reste pris en compte même avec `--no-defaults` (sauf `--no-login-paths`).

### `\s` : vérifier sa connexion

```
\s
```

```
Connection id:          12
Current database:       labdb
Current user:           root@localhost
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Uptime:                 2 hours 15 min 32 sec
```

`Connection: Localhost via UNIX socket` confirme la connexion locale ; `Current database` montre la base active.

## Explorer l'instance

Contrairement à PostgreSQL qui utilise des méta-commandes (`\l`, `\dt`, `\d`), MySQL s'appuie sur des instructions `SHOW` et des vues `INFORMATION_SCHEMA`.

```sql
SHOW DATABASES;
```

| Base | Rôle |
|------|------|
| `information_schema` | Catalogue virtuel, métadonnées sur toutes les tables, colonnes, index |
| `mysql` | Tables système : comptes, privilèges, data dictionary |
| `performance_schema` | Instrumentation interne : requêtes, verrous, I/O, threads |
| `sys` | Vues simplifiées au-dessus de `performance_schema` |
| `labdb` | Base applicative |

```sql
USE labdb;              -- change la base active, plus besoin de préfixer les tables
SHOW TABLES;
SHOW COLUMNS FROM clients;   -- alias : DESC clients / DESCRIBE clients
SHOW CREATE TABLE clients\G  -- DDL exact : moteur, charset, contraintes
SHOW INDEX FROM clients\G    -- index, cardinalité, type
```

`SHOW CREATE TABLE` est la commande la plus utile pour reproduire une table sur un autre serveur ou vérifier sa configuration exacte (moteur InnoDB, charset `utf8mb4`, collation).

### `INFORMATION_SCHEMA` : requêtes sur le catalogue

Standard SQL implémenté par MySQL — les métadonnées sont interrogeables avec du SQL classique.

```sql
-- Taille des bases
SELECT table_schema AS base,
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS taille_mb
FROM information_schema.TABLES
GROUP BY table_schema
ORDER BY taille_mb DESC;

-- Colonnes d'une table
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.COLUMNS
WHERE table_schema = 'labdb' AND table_name = 'clients'
ORDER BY ordinal_position;
```

> [!warning] `table_rows` est une estimation
> La colonne `table_rows` de `INFORMATION_SCHEMA.TABLES` est une estimation pour les tables InnoDB, pas un comptage exact — InnoDB utilise un comptage MVCC trop coûteux à maintenir en temps réel. Pour un décompte précis : `SELECT COUNT(*) FROM table`.

### Performance Schema et `sys` : diagnostic en direct

```sql
SELECT * FROM sys.session\G

-- Requêtes les plus lentes (top 10)
SELECT query, exec_count, avg_latency, rows_examined_avg
FROM sys.statements_with_runtimes_in_95th_percentile
LIMIT 10;
```

> [!tip] `sys` est le meilleur point d'entrée
> Les requêtes directes sur `performance_schema` sont verbeuses. Les vues de `sys` sont conçues pour être lisibles par un humain — à utiliser en premier pour le diagnostic.

### Récapitulatif des commandes d'exploration

| Commande | Description |
|----------|-------------|
| `SHOW DATABASES;` | Lister les bases de données |
| `USE <base>;` | Changer de base active |
| `SHOW TABLES;` | Lister les tables de la base active |
| `SHOW COLUMNS FROM <table>;` | Colonnes, types, clés, défauts |
| `SHOW CREATE TABLE <table>\G` | DDL complet (moteur, charset, contraintes) |
| `SHOW INDEX FROM <table>\G` | Index, cardinalité, type |
| `SHOW VARIABLES LIKE 'pattern';` | Variables de configuration serveur |
| `SHOW GLOBAL STATUS LIKE 'pattern';` | Compteurs de statut serveur |
| `SHOW PROCESSLIST;` | Connexions actives et requêtes en cours |

## Formatage et affichage

Le mode par défaut affiche les résultats en tableau horizontal. Pour les résultats larges (beaucoup de colonnes), le mode vertical `\G` est plus lisible :

```sql
SELECT * FROM clients WHERE id = 1;    -- mode tableau
SELECT * FROM clients WHERE id = 1\G   -- mode vertical
```

> [!warning] `\G` remplace le `;`, ne pas mettre les deux
> Le `\G` termine déjà la requête — un `;` supplémentaire produit une requête vide sur la ligne suivante.

Pour paginer les résultats longs, ou capturer toute la session dans un fichier :

```
pager less -S;
SHOW GLOBAL STATUS;
nopager;

tee /tmp/session-mysql.log
-- vos requêtes ici...
notee
```

| Option | Effet |
|--------|-------|
| `-t` / `--table` | Force le mode tableau (défaut en interactif) |
| `-E` / `--vertical` | Force le mode vertical pour toutes les requêtes |
| `-N` / `--skip-column-names` | Supprime les en-têtes de colonnes |
| `-B` / `--batch` | Mode batch (valeurs séparées par tabulation) |
| `--xml` / `--html` | Sortie en XML / HTML |

## Exécuter du SQL

### Mode batch : `-e` et redirection

```bash
mysql -u root -p labdb -e "SELECT nom, equipe FROM clients;"

# Résultat exploitable par un script : pas de tableau, pas d'en-têtes
mysql -u root -p labdb -BNe "SELECT COUNT(*) FROM clients;"
# 5

# Rediriger un fichier SQL complet
mysql -u root -p labdb < /tmp/rapport.sql
```

| Option | Effet |
|--------|-------|
| `-B` | Mode batch (séparateur tabulation, pas de bordures) |
| `-N` | Pas d'en-têtes de colonnes |
| `-e "..."` | Exécuter la requête et quitter |

### `source` : exécuter un fichier SQL depuis le client

```
source /tmp/01-sample-data.sql
\. /tmp/01-sample-data.sql     -- raccourci équivalent
```

Équivalent MySQL du `\i` de `psql` — le fichier est lu et exécuté ligne par ligne.

## Créer et gérer des objets

```sql
CREATE DATABASE mon_projet CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE mon_projet;

CREATE TABLE utilisateurs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  login VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(200) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

> [!tip] Toujours préciser `ENGINE=InnoDB`
> InnoDB est le moteur par défaut depuis MySQL 5.5, mais l'indiquer explicitement dans le DDL documente l'intention et préserve la compatibilité si le moteur par défaut change en configuration.

### Transactions : `BEGIN`, `COMMIT`, `ROLLBACK`

```sql
BEGIN;
INSERT INTO clients (nom, email, equipe) VALUES ('Test Rollback', 'test@lab.dev', 'QA');
SELECT COUNT(*) AS total FROM clients;   -- 11, visible dans la transaction
ROLLBACK;
SELECT COUNT(*) AS total FROM clients;   -- 10, annulé
```

Par défaut, MySQL exécute chaque requête dans sa propre transaction (mode `autocommit`) :

```sql
SELECT @@autocommit;   -- 1

START TRANSACTION;
-- vos modifications ici...
COMMIT;   -- ou ROLLBACK;
```

> [!warning] Le DDL provoque un `COMMIT` implicite
> `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `TRUNCATE` valident automatiquement toute transaction en cours — un `DROP TABLE` en plein milieu d'une transaction commite déjà tout ce qui précède, sans possibilité de `ROLLBACK` a posteriori. MySQL 8.4 supporte l'*atomic DDL* pour de nombreuses opérations InnoDB (un DDL qui échoue en cours de route est entièrement annulé), mais ce n'est pas du DDL transactionnel à la PostgreSQL.

### Supprimer des objets

```sql
DROP TABLE IF EXISTS utilisateurs;
DROP DATABASE IF EXISTS mon_projet;
```

`IF EXISTS` évite une erreur si l'objet n'existe pas — recommandé dans les scripts.

> [!warning] Pas de `CASCADE` automatique sur `DROP TABLE`
> `DROP TABLE` échoue si d'autres tables ont des clés étrangères vers la table cible. `SET FOREIGN_KEY_CHECKS = 0;` permet de contourner cette vérification, mais uniquement dans des scripts de migration contrôlés — jamais en routine.

## Importer et exporter des données

### `LOAD DATA INFILE` : import depuis un fichier sur le serveur

Méthode la plus rapide pour charger des données massives — le fichier doit être sur le serveur :

```sql
LOAD DATA INFILE '/tmp/clients.csv'
INTO TABLE clients
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(nom, email, equipe);
```

> [!info] `LOAD DATA LOCAL INFILE`
> Variante qui lit le fichier depuis le client plutôt que le serveur — plus pratique, mais doit être autorisée côté serveur (`local_infile=ON`) et côté client (`--local-infile=1`). Pour les imports massifs en production, préférer la version sans `LOCAL` : les données ne transitent pas par le réseau.

### `SELECT ... INTO OUTFILE` : export vers un fichier sur le serveur

```sql
SELECT nom, email, equipe
INTO OUTFILE '/tmp/export_clients.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM clients;
```

Le fichier est créé sur le serveur avec les permissions de l'utilisateur `mysql`.

> [!warning] `secure_file_priv` restreint `LOAD DATA` / `INTO OUTFILE`
> Si la variable pointe vers un répertoire (défaut DEB/RPM : `/var/lib/mysql-files`), les fichiers doivent s'y trouver ; si elle est vide, aucune restriction ; si elle vaut `NULL`, ces opérations sont désactivées. Vérifier avec `SHOW VARIABLES LIKE 'secure_file_priv';`.

### `mysqlimport` et `mysqldump` : wrappers shell

```bash
# mysqlimport : wrapper autour de LOAD DATA INFILE
# le nom de fichier (sans extension) devient le nom de la table
mysqlimport -u root -p --local --fields-terminated-by=',' \
  --lines-terminated-by='\n' --ignore-lines=1 \
  labdb /tmp/clients.csv

# mysqldump : export SQL complet (structure + données)
mysqldump -u root -p labdb > /tmp/labdb_backup.sql
mysqldump -u root -p --no-data labdb > /tmp/labdb_schema.sql   -- structure seule
mysqldump -u root -p labdb clients > /tmp/clients_backup.sql   -- une seule table
```

## MySQL Shell (`mysqlsh`) : l'alternative moderne

Client nouvelle génération fourni par Oracle, avec des fonctionnalités absentes du client `mysql` classique :

| Fonctionnalité | `mysql` (classique) | `mysqlsh` |
|-----------------|----------------------|-----------|
| Mode SQL | Oui | Oui |
| Mode JavaScript / Python | Non | Oui |
| Complétion avancée | Basique | Tables, colonnes, fonctions |
| Export/import parallèle | Non | `util.dumpInstance()` / `util.loadDump()` multi-thread |
| AdminAPI (InnoDB Cluster) | Non | Oui |
| X Protocol | Non | Oui (port 33060) |

```bash
sudo apt install mysql-shell    # Debian/Ubuntu
sudo dnf install mysql-shell    # RHEL/Rocky

mysqlsh --sql -u root -p -h localhost
```

> [!tip] Quand utiliser `mysqlsh` ?
> Pour l'administration quotidienne (`SHOW`, `SELECT`, `INSERT`), le client `mysql` classique suffit. `mysqlsh` devient indispensable pour le dump/restore parallèle sur de grosses bases, l'administration InnoDB Cluster et les scripts d'automatisation Python/JavaScript — `util.dumpInstance()`/`util.loadDump()` ne sont disponibles qu'en modes JavaScript et Python, pas en mode SQL.

## Requêtes d'administration courantes

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW VARIABLES WHERE Variable_name IN ('max_connections', 'wait_timeout', 'interactive_timeout');

SHOW PROCESSLIST;
SHOW FULL PROCESSLIST\G   -- vue complète : requêtes longues, verrous

-- Espace disque par base
SELECT table_schema AS 'Base',
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Taille (Mo)',
       COUNT(*) AS 'Tables'
FROM information_schema.TABLES
GROUP BY table_schema
ORDER BY SUM(data_length + index_length) DESC;
```

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| `ERROR 1045 (28000): Access denied for user 'root'@'localhost'` | Mot de passe incorrect | Vérifier le mot de passe ou le réinitialiser — voir [[MySQL 00 — Installation]] |
| `ERROR 2002 (HY000): Can't connect to local MySQL server through socket` | Service MySQL non démarré ou socket absent | `sudo systemctl start mysql` |
| `ERROR 2003 (HY000): Can't connect to MySQL server on 'host'` | Connexion réseau refusée | Vérifier `bind_address` et le pare-feu |
| `ERROR 1049 (42000): Unknown database 'xxx'` | La base n'existe pas | `SHOW DATABASES;` pour lister les bases disponibles |
| `ERROR 3948 (42000): Loading local data is disabled` | `local_infile` désactivé | `SET GLOBAL local_infile = ON;` puis reconnecter avec `--local-infile=1` |
| Le prompt affiche `    ->` au lieu de `mysql>` | Requête non terminée (`;` manquant) | Taper `;` pour terminer, ou `\c` pour annuler |

## Pour aller plus loin

Ce guide couvre l'administration quotidienne via le client `mysql`. Le dimensionnement fin (buffer pool, connexions, binary log, slow query log), la sécurisation avancée (TLS, rôles, moindre privilège) et la sauvegarde/restauration complète (`mysqldump` avancé, `mysqlsh`, PITR via binlog) restent des guides pratiques annoncés par la ressource source, non encore couverts dans ce vault — voir [[Manques]].

Sources : [Prise en main du client mysql — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/prise-en-main-mysql-cli/)
