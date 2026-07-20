#bdd #mysql #avancé #sauvegarde #pitr

## Les deux familles de sauvegarde

### Sauvegarde logique : export SQL ou CSV

Produit un fichier texte contenant les instructions SQL (`CREATE TABLE`, `INSERT INTO`) pour reconstruire la base — comme photographier un livre page par page pour le réimprimer.

Outils : `mysqldump` (mono-thread, inclus), MySQL Shell `util.dumpInstance()` (multi-thread, compression).

Avantages : portable entre versions, sélectif (une table, un schéma), lisible par un humain. Limites : lent sur les grosses bases, ne permet pas le PITR seul.

### Sauvegarde physique : copie des fichiers InnoDB

Copie les fichiers de données bruts (tablespaces InnoDB, redo logs) — comme cloner le disque dur d'un serveur.

Outil : Percona XtraBackup (open source, sauvegarde à chaud sans verrouillage InnoDB).

Avantages : rapide (copie binaire), à chaud, base pour le PITR physique. Limites : restauration du cluster entier (pas sélectif), pas portable entre versions majeures, outil externe.

> [!info] MySQL Enterprise Backup (MEB)
> Équivalent commercial d'Oracle. XtraBackup est l'alternative open source la plus utilisée, celle détaillée ici.

### Comparatif rapide

| Critère | Logique (`mysqldump`/`mysqlsh`) | Physique (XtraBackup) |
|---------|-----------------------------------|--------------------------|
| Granularité | Table, schéma ou instance | Instance complète |
| PITR | Non seul — nécessite les binlogs | Oui, avec les binlogs |
| Portabilité | Entre versions, entre OS | Même version majeure |
| Vitesse sauvegarde | Lent (SQL sérialisé), `mysqlsh` améliore avec le parallélisme | Rapide (copie fichiers) |
| Vitesse restauration | Lent (réexécute les `INSERT`), `mysqlsh` améliore avec `loadDump` | Rapide (copie fichiers) |
| Impact serveur | Faible avec `--single-transaction` | Faible (copie à chaud) |

> [!tip] Règle pratique
> Commencer par `mysqldump` ou MySQL Shell. Passer à XtraBackup quand la volumétrie ou le RTO l'exige — typiquement au-delà de 50-100 Go.

## Sauvegarde logique avec `mysqldump`

Outil historique, mono-thread, inclus dans l'installation standard.

```bash
# Dump complet de l'instance (y compris bases système)
mysqldump -u root -p --all-databases --single-transaction \
  --routines --triggers --events \
  > /tmp/full_instance.sql

# Dump d'une base spécifique
mysqldump -u root -p --single-transaction \
  --routines --triggers --events \
  lab_mysql > /tmp/lab_mysql.sql

# Une seule table / plusieurs tables / tout sauf certaines tables
mysqldump -u root -p --single-transaction lab_mysql clients > /tmp/clients_only.sql
mysqldump -u root -p --single-transaction lab_mysql clients commandes > /tmp/clients_commandes.sql
mysqldump -u root -p --single-transaction \
  --ignore-table=lab_mysql.logs_acces \
  lab_mysql > /tmp/lab_sans_logs.sql
```

Le fichier produit est du SQL pur, inspectable avec `head`/`grep`.

### Options essentielles

| Option | Rôle | Indispensable ? |
|--------|------|-------------------|
| `--single-transaction` | Démarre une transaction `REPEATABLE READ` pour un dump cohérent sans verrouiller les tables (InnoDB uniquement) | Oui — sans elle, `mysqldump` pose des `LOCK TABLES` qui bloquent les écritures |
| `--routines` | Inclut procédures stockées et fonctions | Oui, souvent oublié |
| `--triggers` | Inclut les triggers | Oui (activé par défaut) |
| `--events` | Inclut le planificateur d'événements | Oui si `EVENT SCHEDULER` utilisé |
| `--set-gtid-purged` | `AUTO` (défaut) ajoute `SET @@GLOBAL.gtid_purged` si GTID activé ; `ON` force ; `OFF` omet ; `COMMENTED` commente | Laisser `AUTO`, sauf restauration sur serveur isolé sans réplication (`OFF`) |
| `--result-file` | Écrit dans un fichier plutôt que stdout (évite les soucis d'encodage Windows) | Optionnel |

> [!warning] `--single-transaction` ne fonctionne qu'avec InnoDB
> Sur des tables MyISAM restantes, `mysqldump` pose quand même un verrou global. Vérifier : `SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'votre_base' AND ENGINE != 'InnoDB';` — voir [[MySQL 04 — Moteurs de stockage (InnoDB vs les autres)]].

### Limites sur les grosses bases

`mysqldump` exporte les tables une par une, séquentiellement : sur 100 Go, la sauvegarde peut prendre plusieurs heures, et la restauration (`mysql < dump.sql`) est encore plus lente (rejeu des `INSERT` un par un), sans compression native. Passer à MySQL Shell dès que les dumps dépassent 30 minutes ou que la base atteint quelques dizaines de Go.

## Sauvegarde avec MySQL Shell (`mysqlsh`)

Les utilitaires de dump/chargement de `mysqlsh` sont multi-threads et compressés — successeur recommandé de `mysqlpump` (déprécié depuis MySQL 8.0.34). Voir [[MySQL 10 — Prise en main du client mysql (CLI)]] pour une présentation générale de `mysqlsh`.

```bash
mysqlsh root@localhost -- util dump-instance /tmp/full_dump
```

```js
// Mode JavaScript dans mysqlsh
util.dumpInstance("/tmp/full_dump", {
  threads: 4,
  compression: "zstd"
})

// Pour une seule base
util.dumpSchemas(["lab_mysql"], "/tmp/lab_dump", {
  threads: 4,
  compression: "zstd"
})
```

Le résultat est un répertoire avec un fichier DDL (`.sql`) et un fichier de données compressé (`.tsv.zst`) par table — cette séparation permet la restauration parallèle et sélective.

| Option | Défaut | Recommandation |
|--------|--------|------------------|
| `threads` | 4 | Nombre de cœurs CPU disponibles, sans dépasser |
| `compression` | `zstd` | Laisser `zstd` — meilleur ratio compression/vitesse |
| `chunking` | `true` | Découpe les grosses tables pour paralléliser l'export |
| `bytesPerChunk` | `256M` | Diminuer si la RAM est limitée |

La combinaison parallélisme + compression rend MySQL Shell 5 à 10 fois plus rapide que `mysqldump` sur les bases volumineuses.

```js
util.loadDump("/tmp/full_dump", {
  threads: 4,
  resetProgress: true,
  ignoreExistingObjects: false
})
```

> [!tip] `loadDump()` reprend sur erreur
> La progression est enregistrée dans `load-progress*.json`. Si la restauration est interrompue, relancer la même commande (sans `resetProgress: true`) reprend là où elle s'est arrêtée.

### `mysqlpump` est déprécié

| Ancien (`mysqlpump`) | Nouveau (`mysqlsh`) |
|------------------------|------------------------|
| `mysqlpump --all-databases` | `util.dumpInstance("/path/dump")` |
| `mysqlpump --databases db1 db2` | `util.dumpSchemas(["db1","db2"], "/path/dump")` |
| `mysqlpump --default-parallelism=4` | `util.dumpInstance("/path/dump", {threads: 4})` |

## Sauvegarde physique avec Percona XtraBackup

Copie les fichiers de données InnoDB pendant que le serveur tourne, sans verrouillage des tables (sauf un bref `FTWRL` pour les métadonnées non-InnoDB) — équivalent MySQL de `pg_basebackup`.

Trois étapes : **Backup** (copie fichiers InnoDB + capture continue du redo log) → **Prepare** (applique le redo log pour rendre le backup cohérent, comme un crash recovery) → **Restore** (copie le backup préparé à la place du datadir).

> [!warning] La version de XtraBackup doit correspondre à la version majeure de MySQL
> XtraBackup 8.4 pour MySQL 8.4 — XtraBackup 8.0 ne fonctionne pas avec MySQL 8.4.

```bash
# Installation Debian/Ubuntu
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
sudo percona-release setup pxb-84
sudo apt update && sudo apt install percona-xtrabackup-84

# Installation RHEL/Rocky
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo percona-release setup pxb-84
sudo yum install percona-xtrabackup-84
```

```bash
# Sauvegarde complète
sudo xtrabackup --backup \
  --user=root --password='VotreMotDePasse' \
  --target-dir=/var/backups/mysql/full_$(date +%Y%m%d)

# Sauvegarde incrémentale (pages modifiées depuis la full)
sudo xtrabackup --backup \
  --user=root --password='VotreMotDePasse' \
  --target-dir=/var/backups/mysql/incr_$(date +%Y%m%d) \
  --incremental-basedir=/var/backups/mysql/full_20260413
```

Le message `completed OK!` confirme la réussite.

### Préparation et restauration

```bash
# 1. Préparer la full (applique le redo log)
sudo xtrabackup --prepare --target-dir=/var/backups/mysql/full_20260413

# 2. Si incrémentale, l'appliquer sur la full
sudo xtrabackup --prepare \
  --target-dir=/var/backups/mysql/full_20260413 \
  --incremental-dir=/var/backups/mysql/incr_20260414

# 3. Arrêter MySQL
sudo systemctl stop mysql

# 4. Remplacer le data directory
sudo mv /var/lib/mysql /var/lib/mysql_damaged
sudo xtrabackup --copy-back --target-dir=/var/backups/mysql/full_20260413
sudo chown -R mysql:mysql /var/lib/mysql

# 5. Redémarrer MySQL
sudo systemctl start mysql
```

> [!warning] Permissions après restauration
> Après `--copy-back`, les fichiers appartiennent à `root`. Sans `chown -R mysql:mysql /var/lib/mysql`, MySQL refuse de démarrer (erreur de permission sur les tablespaces).

## Archivage des binary logs

En MySQL 8.4, le binary log est activé par défaut (`log_bin = ON`) :

```sql
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'log_bin_basename';
SHOW BINARY LOGS;
```

Configuration importante (voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]) :

```ini
[mysqld]
binlog_expire_logs_seconds = 2592000   # rétention : 30 jours par défaut
binlog_format = ROW                     # cohérence (défaut 8.4)
sync_binlog = 1                         # flush à chaque commit
```

> [!info] Différence avec PostgreSQL
> Les binary logs combinent le rôle des WAL de PostgreSQL pour la réplication et le PITR, mais restent un mécanisme distinct du redo log InnoDB (crash recovery interne) — voir [[MySQL 07 — Binary log — réplication & PITR]]. PostgreSQL n'a qu'un seul mécanisme pour les deux.

### `mysqlbinlog` : lire et filtrer les événements

```bash
# Lire un binlog complet
mysqlbinlog /var/lib/mysql/binlog.000002

# Filtrer par base et par période
mysqlbinlog /var/lib/mysql/binlog.000002 \
  --database=lab_mysql \
  --start-datetime="2026-04-13 10:00:00" \
  --stop-datetime="2026-04-13 10:20:00" \
  --verbose
```

| Option | Rôle |
|--------|------|
| `--database` | Filtrer par base (ne fonctionne pas pour toutes les requêtes cross-db) |
| `--start-datetime` / `--stop-datetime` | Filtrer par date |
| `--start-position` / `--stop-position` | Filtrer par position dans le binlog |
| `--verbose` (`-v`) | Décoder les événements ROW en pseudo-SQL lisible |
| `--base64-output=DECODE-ROWS` | Afficher les données sans le base64 brut |

### Copier les binlogs vers un stockage externe

```bash
cp /var/lib/mysql/binlog.* /mnt/backup/mysql-binlogs/
rsync -avz /var/lib/mysql/binlog.* backup-server:/backups/mysql-binlogs/
```

> [!warning] Ne pas copier les binlogs pendant une écriture active
> `FLUSH BINARY LOGS;` force la rotation avant de copier — le binlog en cours d'écriture n'est pas garanti cohérent. Copier ensuite uniquement les fichiers précédents, pas le dernier actif.

Pour un archivage en continu (équivalent d'un `tail -f` sur les binlogs) :

```bash
mysqlbinlog --read-from-remote-server --host=localhost \
  --user=repl_user --password='xxx' \
  --raw --stop-never \
  binlog.000001 \
  --result-file=/mnt/backup/mysql-binlogs/
```

## Restauration Point-In-Time (PITR)

Capacité à restaurer la base à un instant précis — le filet de sécurité contre un `DROP TABLE` accidentel. Combine un **dump de référence** (point de départ) et le **rejeu des binlogs** entre le dump et l'instant cible :

```
[Dump à 02h00] ──── binlogs ──── [DROP TABLE à 14h32] ──── binlogs ── [maintenant]
                    │                                │
                    On rejoue ces binlogs             On s'arrête AVANT cette position
```

### Démonstration : simuler un `DROP TABLE` et restaurer

```bash
# 1. Dump de référence — --flush-logs force une rotation, --source-data=2
#    inscrit la position binlog en commentaire (--master-data est un alias déprécié)
mysqldump -u root -p --single-transaction \
  --routines --triggers --events \
  --flush-logs --source-data=2 \
  lab_mysql > /tmp/lab_mysql_backup.sql

head -30 /tmp/lab_mysql_backup.sql | grep "CHANGE"
# -- CHANGE REPLICATION SOURCE TO SOURCE_LOG_FILE='binlog.000004', SOURCE_LOG_POS=157;
```

```sql
-- 2. Insérer une ligne qui doit survivre à la restauration
INSERT INTO clients (nom, email, ville) VALUES ('PITR Test', 'pitr@example.com', 'Nantes');

-- 3. Noter le safe point
SELECT NOW() AS safe_timestamp;   -- 2026-04-13 10:19:49

-- 4. Simuler la catastrophe
DROP TABLE commandes;
DROP TABLE clients;
```

```sql
-- 5. Forcer la rotation pour isoler le DROP dans un binlog clos
FLUSH BINARY LOGS;
SHOW BINARY LOGS;   -- le DROP est dans binlog.000004, la suite dans binlog.000005
```

```bash
# 6. Restaurer le dump de référence
mysql -u root -p lab_mysql < /tmp/lab_mysql_backup.sql

# 7. Rejouer les binlogs depuis la position du dump jusqu'au safe point (avant le DROP)
mysqlbinlog /var/lib/mysql/binlog.000004 \
  --start-position=157 \
  --stop-datetime="2026-04-13 10:19:49" \
  --database=lab_mysql \
  | mysql -u root -p lab_mysql
```

```sql
-- 8. Vérifier : 5 clients d'origine + "PITR Test" = 6, sans DROP
SELECT nom, email FROM clients ORDER BY id;
```

### Identifier le point de restauration

Par date/heure (le plus courant) :

```bash
mysqlbinlog binlog.000004 --stop-datetime="2026-04-13 10:19:49"
```

Par position (plus précis quand plusieurs transactions partagent la même seconde) :

```bash
mysqlbinlog binlog.000004 --verbose | grep -B5 "DROP TABLE"
# repérer la position juste avant, ex. end_log_pos 12639 → DROP TABLE

mysqlbinlog binlog.000004 --start-position=157 --stop-position=12534 \
  | mysql -u root -p lab_mysql
```

Par GTID, si activé (recommandé en environnement répliqué) :

```bash
mysqlbinlog binlog.000004 \
  --exclude-gtids="7c1b9e1a-...:15" \
  | mysql -u root -p lab_mysql
```

### Vérifier la cohérence après restauration

```sql
SET FOREIGN_KEY_CHECKS = 1;

SELECT (SELECT COUNT(*) FROM clients) AS nb_clients,
       (SELECT COUNT(*) FROM commandes) AS nb_commandes;

CHECK TABLE clients, commandes;
```

## Stratégie de sauvegarde recommandée

| Taille de base | Outils | RPO | RTO |
|-----------------|--------|-----|-----|
| < 10 Go | `mysqldump` quotidien (`--single-transaction --routines --triggers --events --flush-logs --source-data=2`) + binlogs archivés en continu, test hebdomadaire, rétention 7 dumps + 1 mensuel | Quelques minutes | Quelques minutes |
| 10-500 Go | MySQL Shell `util.dumpInstance()` quotidien (parallèle + zstd) + binlogs continus, full hebdo + incrémentaux quotidiens, test PITR mensuel | Quelques minutes | 15-60 minutes |
| > 500 Go | XtraBackup complète hebdo + incrémentale quotidienne, binlogs streamés (`mysqlbinlog --read-from-remote-server`), dumps sélectifs `mysqlsh` pour tables critiques, test automatisé en CI | Quelques secondes | 30 min à quelques heures |

## Tester ses sauvegardes

Une sauvegarde non testée n'est pas une sauvegarde.

| Niveau | Méthode | Ce que ça prouve |
|--------|---------|--------------------|
| 1. Intégrité fichier | Vérifier que le dump n'est pas vide ou tronqué | Le fichier a été produit sans erreur |
| 2. Restauration | Restaurer dans une base jetable | Le dump est lisible et complet |
| 3. Cohérence | Requêtes de contrôle sur la base restaurée | Les données sont exploitables |

```bash
#!/bin/bash
set -euo pipefail

DUMP="/var/backups/mysql/lab_mysql_$(date +%Y%m%d).sql"
TEST_DB="verify_$(date +%Y%m%d)"

mysqldump -u root -p"${MYSQL_ROOT_PASSWORD}" \
  --single-transaction --routines --triggers --events \
  lab_mysql > "$DUMP"

if [ ! -s "$DUMP" ]; then
  echo "ERREUR: dump vide" >&2; exit 1
fi

mysql -u root -p"${MYSQL_ROOT_PASSWORD}" -e "CREATE DATABASE $TEST_DB;"
mysql -u root -p"${MYSQL_ROOT_PASSWORD}" "$TEST_DB" < "$DUMP"

COUNT=$(mysql -u root -p"${MYSQL_ROOT_PASSWORD}" -N -e \
  "SELECT COUNT(*) FROM $TEST_DB.clients;")

if [ "$COUNT" -gt 0 ]; then
  echo "OK: $COUNT clients restaurés"
else
  echo "ERREUR: base vide après restauration" >&2
  exit 1
fi

mysql -u root -p"${MYSQL_ROOT_PASSWORD}" -e "DROP DATABASE $TEST_DB;"
```

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| `mysqldump: Got error: 1044: Access denied` | Utilisateur sans privilège `LOCK TABLES`/`SELECT` | `GRANT SELECT, LOCK TABLES, SHOW VIEW, EVENT, TRIGGER ON *.* TO 'backup_user'@'localhost';` |
| `ERROR 1045 (28000): LOCK TABLES` en plein dump | Tables MyISAM bloquant le verrou global | `--single-transaction` (InnoDB uniquement) ou convertir en InnoDB |
| Dump très lent (> 1h pour < 50 Go) | `mysqldump` est mono-thread | Passer à MySQL Shell `util.dumpInstance()` avec `threads: 4` |
| `ERROR: Unknown table 'COLUMN_STATISTICS'` à la restauration sur un ancien MySQL | `--column-statistics` activé par défaut dans `mysqldump` 8.x | Ajouter `--column-statistics=0` au dump |
| `mysqlbinlog: ERROR: Could not find GTID state` | Binlogs purgés avant l'archivage | Vérifier `binlog_expire_logs_seconds`, augmenter la rétention |
| `Got a packet bigger than 'max_allowed_packet'` | Ligne/BLOB trop volumineux | `--max-allowed-packet=512M` au dump et à la restauration |
| XtraBackup : `log block numbers mismatch` | Version XtraBackup incompatible | XtraBackup 8.4 obligatoire pour MySQL 8.4 |
| PITR : données manquantes | `--stop-datetime` trop tôt ou binlog manquant | Vérifier le timestamp exact avec `mysqlbinlog --verbose` |

## Pour aller plus loin

La réplication (GTID, source-réplica, Group Replication, InnoDB Cluster) est couverte à partir de [[MySQL 26 — Concepts de réplication & GTID]], le monitoring (Performance Schema, slow queries, `OPTIMIZE TABLE`) à partir de [[MySQL 13 — Observer l'activité (PROCESSLIST & connexions)]], et la sécurisation avancée (authentification, rôles, TLS, audit) à partir de [[MySQL 21 — Authentification (caching_sha2_password & authentication_policy)]].

Sources : [Sauvegarder et restaurer MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/sauvegarde-restauration/)
