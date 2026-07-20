#bdd #mysql #pièges #erreurs #debugging

## 🪤 Piège 1 — Modifier la base `mysql` directement

```sql
-- ❌ Peut corrompre le cache des privilèges
INSERT INTO mysql.user (...) VALUES (...);
```

```sql
-- ✅ Toujours passer par les commandes officielles
CREATE USER 'nouvel_utilisateur' WITH PASSWORD 'mot_de_passe';
GRANT SELECT, INSERT ON labdb.* TO 'nouvel_utilisateur';
```

> [!warning] Le cache des privilèges ne se synchronise pas tout seul
> Voir [[MySQL 02 — Bases de données, schémas & bases système]].

---

## 🪤 Piège 2 — Garder des tables MyISAM en production

```sql
-- ❌ Pas de transactions, pas de crash recovery, corruption possible
CREATE TABLE commandes (...) ENGINE=MyISAM;
```

```sql
-- ✅ Migrer vers InnoDB
ALTER TABLE commandes ENGINE=InnoDB;
```

> [!warning] Une coupure de courant peut corrompre une table MyISAM de façon irrécupérable
> Voir [[MySQL 04 — Moteurs de stockage (InnoDB vs les autres)]].

---

## 🪤 Piège 3 — Désactiver le doublewrite buffer

```
❌ innodb_doublewrite = OFF sur un système de fichiers ext4/XFS classique
```

> [!warning] Risque de corruption réel sans cette protection
> Seuls des filesystems à écriture atomique garantie (ZFS, certains SAN) permettent de s'en passer. Voir [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]].

---

## 🪤 Piège 4 — Copier `auto.cnf` en clonant un serveur

```bash
# ❌ Le clone garde le même server_uuid que l'original
cp -r /var/lib/mysql/ /var/lib/mysql-clone/
```

```bash
# ✅ Supprimer auto.cnf avant le premier démarrage du clone
rm /var/lib/mysql-clone/auto.cnf
```

> [!warning] Deux serveurs avec le même UUID cassent la réplication
> Voir [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]].

---

## 🪤 Piège 5 — Utiliser `STATEMENT` comme format de binlog avec des fonctions non déterministes

```sql
-- ❌ NOW() est rejoué à un instant différent sur le réplica
UPDATE sessions SET expire_at = NOW() + INTERVAL 1 HOUR;
```

> [!warning] Une réplication silencieusement désynchronisée
> `ROW` (format par défaut depuis MySQL 5.7.7) élimine ce risque en enregistrant le résultat plutôt que la requête. Voir [[MySQL 07 — Binary log — réplication & PITR]].

---

## 🪤 Piège 6 — Ne pas dimensionner le buffer pool

```sql
-- ❌ Valeur par défaut (128 Mo) conservée sur un serveur de production dédié
SHOW VARIABLES LIKE 'innodb_buffer_pool_size'; -- 134217728
```

> [!warning] MySQL va constamment chercher sur disque au lieu du cache
> Viser 50 à 80% de la RAM disponible sur un serveur dédié à MySQL. Voir [[MySQL 05 — InnoDB — le buffer pool]].

---

## 🪤 Piège 7 — Installer MySQL et MariaDB sur la même machine

```bash
# ❌ Conflit de paquets : même port 3306, même socket, mêmes noms de fichiers
sudo apt install mysql-server   # alors que MariaDB est déjà installé
```

> [!warning] Désinstaller complètement l'un avant d'installer l'autre
> Voir [[MySQL 00 — Installation]] pour la procédure de désinstallation propre de MariaDB — la coexistence des deux n'est pas une option.

---

## 🪤 Piège 8 — Ignorer le mot de passe root temporaire sur RHEL/Rocky

```bash
# ❌ Tenter mysql -u root -p sans avoir récupéré le mot de passe généré
mysql -u root -p
```

```bash
# ✅ D'abord récupérer le mot de passe temporaire
sudo grep 'temporary password' /var/log/mysqld.log
```

> [!warning] Contrairement à Debian, RHEL ne laisse jamais le mot de passe vide
> Sur RHEL/Rocky, un mot de passe temporaire est systématiquement généré au premier démarrage — l'ignorer est la cause la plus fréquente de blocage juste après l'installation. Voir [[MySQL 00 — Installation]].

---

## 🪤 Piège 9 — Compter sur `ROLLBACK` après un DDL

```sql
-- ❌ Le DROP valide déjà tout ce qui précède dans la transaction
BEGIN;
UPDATE clients SET equipe = 'QA' WHERE id = 1;
DROP TABLE ancienne_table;
ROLLBACK;  -- ne annule PAS le UPDATE : déjà commité par le DROP
```

> [!warning] Le DDL provoque un `COMMIT` implicite
> `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `TRUNCATE` valident automatiquement toute transaction en cours. MySQL 8.4 supporte l'atomic DDL pour certaines opérations InnoDB, mais ce n'est pas du DDL transactionnel à la PostgreSQL — un `ROLLBACK` après un DDL n'annule jamais ce DDL ni ce qui le précédait. Voir [[MySQL 10 — Prise en main du client mysql (CLI)]].

---

## 🪤 Piège 10 — Passer le mot de passe collé à `-p`

```bash
# ❌ Visible dans ps aux et dans l'historique shell
mysql -u root -pMotDePasseFort!2026
```

```bash
# ✅ -p seul déclenche une invite interactive sécurisée
mysql -u root -p
```

> [!warning] Un mot de passe en argument de commande n'est jamais secret
> Tout utilisateur du système peut le lire via `ps aux` pendant l'exécution, ou via l'historique du shell ensuite. Voir [[MySQL 10 — Prise en main du client mysql (CLI)]].

---

## 🪤 Piège 11 — Éditer `mysqld-auto.cnf` à la main

```bash
# ❌ Une virgule oubliée dans ce JSON empêche mysqld de démarrer
vim /var/lib/mysql/mysqld-auto.cnf
```

```sql
-- ✅ Toujours passer par SET PERSIST / RESET PERSIST
SET PERSIST innodb_buffer_pool_size = 1073741824;
RESET PERSIST slow_query_log;
```

> [!warning] `mysqld-auto.cnf` est un fichier JSON généré par MySQL
> Une syntaxe cassée empêche le serveur de redémarrer, sans message d'erreur clair côté fichier lui-même. Voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]].

---

## 🪤 Piège 12 — Oublier `general_log` activé en production

```sql
-- ❌ Activé pour du debug, jamais désactivé
SET GLOBAL general_log = ON;
```

> [!warning] Volume de logs et impact performance
> Le general log trace absolument toutes les requêtes — laissé actif en production, il fait exploser l'espace disque et dégrade les performances. Toujours le repasser à `OFF` après le debug, et ne jamais le persister avec `SET PERSIST`. Voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]].

---

## 🪤 Piège 13 — Ouvrir `bind_address` avant de sécuriser les comptes

```ini
# ❌ Réseau ouvert alors que des comptes ont encore des mots de passe faibles ou 'root'@'%'
[mysqld]
bind_address = 0.0.0.0
```

> [!warning] `bind_address` expose l'instance dès le redémarrage
> Ouvrir l'écoute réseau avant d'avoir revu les comptes et privilèges (voir [[Manques]], guide Sécurisation) expose immédiatement toute connexion mal protégée à un accès distant. Voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]].

---

## 🪤 Piège 14 — `mysqldump` sans `--single-transaction` sur une base InnoDB active

```bash
# ❌ Pose un LOCK TABLES global, bloque toutes les écritures pendant le dump
mysqldump -u root -p lab_mysql > /tmp/lab_mysql.sql
```

```bash
# ✅ Dump cohérent sans verrouillage sur InnoDB
mysqldump -u root -p --single-transaction lab_mysql > /tmp/lab_mysql.sql
```

> [!warning] Une production bloquée par une sauvegarde
> Sans `--single-transaction`, `mysqldump` verrouille les tables le temps du dump — inacceptable sur une base InnoDB en écriture active. Voir [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]].

---

## 🪤 Piège 15 — Copier le binlog en cours d'écriture

```bash
# ❌ Le fichier actif n'est pas garanti cohérent au moment de la copie
cp /var/lib/mysql/binlog.000005 /mnt/backup/
```

```sql
-- ✅ Forcer la rotation avant de copier les fichiers précédents
FLUSH BINARY LOGS;
```

> [!warning] Un PITR basé sur un binlog tronqué est inutilisable
> Copier uniquement les fichiers clos par la rotation, jamais le binlog actif. Voir [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]].

---

## 🪤 Piège 16 — Une sauvegarde jamais restaurée pour de vrai

```bash
# ❌ Le cron tourne depuis des mois, personne n'a testé la restauration
0 2 * * * mysqldump -u root -p... --all-databases > /backup/full.sql
```

> [!warning] Un backup non testé est un faux sentiment de sécurité
> Un dump vide, tronqué, ou produit avec un utilisateur sans les bons privilèges passe inaperçu tant qu'aucune restauration réelle n'est tentée. Automatiser un test de restauration régulier (voir script dans [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]]).

---

## 🪤 Piège 17 — Lancer OPTIMIZE TABLE à l'aveugle sur une table de production

```sql
-- ❌ Reconstruction complète (copie temporaire) sans avoir mesuré le besoin réel
OPTIMIZE TABLE lab_mysql.logs;
```

> [!warning] Une opération lourde, à réserver aux heures creuses
> `OPTIMIZE TABLE` sur InnoDB équivaut à `ALTER TABLE ... FORCE` + `ANALYZE TABLE` : une copie temporaire de toute la table, nécessitant un espace disque équivalent à sa taille. Vérifier d'abord l'espace réellement récupérable et planifier hors pic. Voir [[MySQL 19 — Maintenance des tables]].

---

## 🪤 Piège 18 — Ajouter un index sans avoir lu le plan d'exécution

```sql
-- ❌ Index ajouté "au cas où", sans confirmer qu'il cible le bon problème
CREATE INDEX idx_logs_date ON logs (created_at);
```

> [!warning] Un index non vérifié peut aggraver la situation
> Toujours confirmer avec `EXPLAIN`/`EXPLAIN ANALYZE` (voir [[MySQL 17 — EXPLAIN & EXPLAIN ANALYZE]]) que la colonne indexée est bien celle qui cause un `type = ALL` avant de créer l'index — un index inutile ralentit les écritures sans accélérer la requête visée.

---

## 🪤 Piège 19 — Créer un compte applicatif avec `@'%'`

```sql
-- ❌ Accessible depuis n'importe quelle IP
CREATE USER 'app_writer'@'%' IDENTIFIED BY 'AppWriter2026!';
```

> [!warning] L'équivalent d'un `0.0.0.0/0` en pare-feu
> `@'%'` autorise la connexion depuis n'importe quelle IP — l'équivalent d'un `host all all 0.0.0.0/0` dans `pg_hba.conf` côté PostgreSQL. Restreindre systématiquement à un sous-réseau (`'10.0.1.%'`) ou une IP précise. Voir [[MySQL 22 — Gestion des utilisateurs, verrouillage & validate_password]].

---

## 🪤 Piège 20 — Accorder un rôle sans l'activer

```sql
GRANT 'role_reader' TO 'monitoring'@'10.0.1.50';
-- Puis, connecté en tant que monitoring :
SELECT * FROM lab_mysql.logs;
-- ERROR 1142 (42000): SELECT command denied
```

> [!warning] Un rôle accordé n'est pas actif par défaut
> `GRANT role TO user` attribue le rôle mais ne l'active pas à la connexion. Vérifier avec `SELECT CURRENT_ROLE();` — si `NONE`, exécuter `SET DEFAULT ROLE ALL TO user@host;` ou activer globalement `SET PERSIST activate_all_roles_on_login = ON;`. Voir [[MySQL 23 — Rôles, privilèges granulaires & moindre privilège]].

---

## 🪤 Piège 21 — Confondre REQUIRE SSL (serveur) et --ssl-mode (client)

```sql
-- Le compte force le chiffrement...
ALTER USER 'app_secure'@'10.0.1.%' REQUIRE SSL;
```

```bash
# ...mais le client ne vérifie pas l'identité du serveur
mysql -u app_secure -h 10.0.1.5 -p --ssl-mode=REQUIRED
```

> [!warning] `REQUIRE SSL` chiffre, il ne vérifie pas l'identité du serveur
> `REQUIRE SSL` (côté serveur, dans `CREATE USER`) garantit uniquement que le transport est chiffré. Sans `--ssl-mode=VERIFY_CA` ou `VERIFY_IDENTITY` côté client, la connexion reste vulnérable à une attaque *man-in-the-middle* avec un faux serveur. Voir [[MySQL 24 — Chiffrement TLS]].

---

## 🪤 Piège 22 — Croire qu'un seul replica semi-synchrone garantit toujours le RPO=0

```sql
SHOW STATUS LIKE 'Rpl_semi_sync_source_status';  -- ON... jusqu'à ce que le replica tombe
```

> [!warning] Le fallback en asynchrone est silencieux
> Si l'unique replica semi-synchrone devient injoignable, le source bascule automatiquement en asynchrone après `rpl_semi_sync_source_timeout` — sans erreur bloquante ni alerte visible. La garantie RPO=0 disparaît sans notification. Utiliser au moins deux replicas semi-synchrones en production. Voir [[MySQL 30 — Semi-synchrone, Group Replication & InnoDB Cluster]].

---

## 🪤 Piège 23 — Utiliser GET_SOURCE_PUBLIC_KEY = 1 en production sans TLS

```sql
-- ❌ Récupère la clé publique à la volée, exposé au MITM sur la première connexion
CHANGE REPLICATION SOURCE TO ..., GET_SOURCE_PUBLIC_KEY = 1;
```

> [!warning] Préférer TLS ou une clé publique locale
> `GET_SOURCE_PUBLIC_KEY = 1` expose à une attaque *man-in-the-middle* lors du premier échange de clé. En production, utiliser `SOURCE_SSL = 1` ou fournir la clé publique localement avec `SOURCE_PUBLIC_KEY_PATH`. Voir [[MySQL 27 — Mettre en place une réplication source-replica (GTID)]].

---

## 🪤 Piège 24 — Exécuter une transaction directement sur un replica

```sql
-- Sur le replica, en contournant read_only via un compte SUPER
SET GLOBAL super_read_only = OFF;
INSERT INTO lab_mysql.clients (nom) VALUES ('Erreur');
```

> [!warning] Une transaction « errante » casse la cohérence GTID
> Une transaction exécutée directement sur un replica crée un GTID que le source ne connaît pas (*errant transaction*). Elle peut bloquer un futur failover ou une reconfiguration en source. Toujours garder `read_only` **et** `super_read_only` activés sur un replica — voir [[MySQL 28 — Surveiller la réplication & calculer le lag]]. En cas d'incident, identifier l'écart avec `GTID_SUBTRACT(replica_gtid, source_gtid)`.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Modification directe de la base `mysql` | `CREATE USER`/`GRANT`/`REVOKE` uniquement |
| Tables MyISAM en production | `ALTER TABLE ... ENGINE=InnoDB` |
| Doublewrite buffer désactivé sur ext4/XFS | Le laisser activé, sauf filesystem à écriture atomique garantie |
| `auto.cnf` copié sur un clone | Le supprimer avant le premier démarrage du clone |
| Binlog `STATEMENT` avec fonctions non déterministes | Utiliser `ROW` (par défaut) |
| Buffer pool laissé à sa valeur par défaut (128 Mo) | Dimensionner à 50-80% de la RAM disponible |
| MySQL et MariaDB installés ensemble | Désinstaller complètement l'un avant l'autre |
| Mot de passe root temporaire RHEL ignoré | Le récupérer dans `/var/log/mysqld.log` avant la première connexion |
| `ROLLBACK` attendu après un DDL | Le DDL commite déjà tout — pas de retour en arrière possible |
| Mot de passe collé à `-p` sur la ligne de commande | Toujours `-p` seul, saisie interactive |
| Édition manuelle de `mysqld-auto.cnf` | `SET PERSIST` / `RESET PERSIST` uniquement |
| `general_log` oublié activé en production | Le repasser à `OFF` après debug, ne jamais le persister |
| `bind_address` ouvert avant sécurisation des comptes | Sécuriser les comptes/privilèges avant d'ouvrir le réseau |
| `mysqldump` sans `--single-transaction` sur InnoDB actif | Toujours l'utiliser pour éviter le verrouillage global |
| Binlog actif copié directement | `FLUSH BINARY LOGS;` avant de copier, jamais le fichier en cours |
| Sauvegarde jamais restaurée pour de vrai | Automatiser un test de restauration régulier |
| `OPTIMIZE TABLE` lancé à l'aveugle en production | Vérifier le besoin réel, planifier hors pic |
| Index ajouté sans lire le plan d'exécution | Confirmer avec `EXPLAIN`/`EXPLAIN ANALYZE` d'abord |
| Compte applicatif créé avec `@'%'` | Restreindre à un sous-réseau ou une IP précise |
| Rôle accordé (`GRANT`) mais non activé | `SET DEFAULT ROLE ALL` ou `activate_all_roles_on_login = ON` |
| `REQUIRE SSL` sans `--ssl-mode=VERIFY_CA` côté client | Ajouter la vérification côté client contre le MITM |
| Un seul replica semi-synchrone considéré comme RPO=0 garanti | En prévoir au moins deux, surveiller le fallback silencieux |
| `GET_SOURCE_PUBLIC_KEY = 1` en production | Utiliser `SOURCE_SSL = 1` ou `SOURCE_PUBLIC_KEY_PATH` |
| Écriture directe sur un replica (transaction errante) | Garder `read_only` + `super_read_only` activés en permanence |
