#bdd #mysql #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Instance** | Un processus `mysqld` en cours d'exécution, avec son datadir et sa configuration propres. |
| **Base de données / Schéma** | Un espace de noms contenant des tables — les deux termes sont strictement synonymes dans MySQL, contrairement à PostgreSQL. |
| **InnoDB** | Moteur de stockage par défaut depuis MySQL 5.5, transactionnel, crash-safe, avec MVCC et verrouillage au niveau ligne. |
| **Buffer pool** | Cache mémoire principal d'InnoDB pour les pages de données et d'index (16 Ko par page). |
| **Redo log** | Journal de crash recovery d'InnoDB, écrit avant toute modification de données (Write-Ahead Logging). |
| **Binary log (binlog)** | Journal de réplication et de PITR, enregistrant les événements DML/DDL — distinct et complémentaire du redo log. |
| **GTID** | *Global Transaction Identifier* — identifiant unique de transaction utilisé pour la réplication. |
| **Data dictionary** | Métadonnées des tables stockées dans des tables InnoDB transactionnelles depuis MySQL 8.0, remplaçant les anciens fichiers `.frm`. |
| **Tablespace** | Conteneur logique de fichiers de données InnoDB — `ibdata1` pour le tablespace système, un `.ibd` par table en fichier-par-table. |
| **Doublewrite buffer** | Protection contre les écritures partielles lors de l'écriture des pages modifiées sur disque. |
| **Change buffer** | Cache des modifications d'index secondaires non encore chargées en mémoire, appliquées lors de leur prochaine lecture. |
| **LTS (Long Term Support)** | Track de versions MySQL à support long (actuellement 8.4), recommandé pour la production stable. |
| **Track Innovation** | Track de versions MySQL (9.x) au cycle plus court, pour un accès plus rapide aux nouveautés. |
| **Thread de connexion** | Thread dédié à un client connecté, à l'intérieur de l'unique processus `mysqld`. |
| **Thread cache** | Réserve de threads recyclés à la déconnexion, évitant de recréer un thread à chaque nouvelle connexion. |
| **Performance Schema** | Système d'instrumentation intégré exposant les métriques internes de l'instance (requêtes, verrous, I/O, threads). |
| **MVCC** | *Multi-Version Concurrency Control* — mécanisme permettant des lectures non bloquantes malgré des écritures concurrentes. |
| **Two-phase commit (XA)** | Mécanisme garantissant la cohérence entre redo log et binary log lors d'une transaction. |
| **MariaDB** | Fork de MySQL créé en 2009 après le rachat par Oracle — partage le protocole réseau et l'essentiel de la syntaxe SQL, mais non interchangeable en production (Galera Cluster vs Group Replication, `mysql_native_password` par défaut). |
| **`mysql_secure_installation`** | Script interactif appliquant les premières mesures de sécurité après installation : suppression des comptes anonymes et de la base `test`, restriction de l'accès root distant. |
| **`auth_socket`** | Méthode d'authentification locale basée sur l'utilisateur du système d'exploitation, utilisée quand aucun mot de passe root n'est défini à l'installation Debian/Ubuntu. |
| **`caching_sha2_password`** | Plugin d'authentification par défaut depuis MySQL 8.0, basé sur SHA-256 avec cache côté serveur — remplace l'ancien `mysql_native_password`. |
| **`--skip-grant-tables`** | Option de démarrage désactivant toute vérification d'authentification, utilisée uniquement pour réinitialiser un mot de passe root perdu. |
| **`debconf-set-selections`** | Mécanisme Debian permettant de pré-configurer les réponses d'un installateur (ex. le mot de passe root MySQL) pour une installation non interactive. |
| **`INFORMATION_SCHEMA`** | Base virtuelle standard SQL exposant les métadonnées (tables, colonnes, index) sous forme de vues interrogeables en SQL classique. |
| **`sys`** | Base fournissant des vues simplifiées et lisibles au-dessus de `performance_schema`, à privilégier pour le diagnostic. |
| **`login-path`** | Profil de connexion chiffré stocké dans `~/.mylogin.cnf` via `mysql_config_editor`, évitant d'exposer le mot de passe en clair. |
| **`autocommit`** | Mode par défaut où chaque requête s'exécute dans sa propre transaction, validée immédiatement — désactivable avec `START TRANSACTION`. |
| **`secure_file_priv`** | Variable serveur restreignant les répertoires autorisés pour `LOAD DATA INFILE` et `SELECT ... INTO OUTFILE`. |
| **`mysqlsh` (MySQL Shell)** | Client nouvelle génération d'Oracle, avec modes SQL/JavaScript/Python, complétion avancée et utilitaires de dump/restore parallèles (`util.dumpInstance()`). |
| **`SET PERSIST`** | Modifie une variable dynamique à chaud et l'écrit dans `mysqld-auto.cnf`, la rendant persistante au redémarrage. |
| **`mysqld-auto.cnf`** | Fichier JSON (`/var/lib/mysql/`) écrit par `SET PERSIST`, prioritaire sur `my.cnf`/`mysqld.cnf` — ne jamais l'éditer à la main. |
| **Variable dynamique / statique** | Dynamique : modifiable à chaud (`SET GLOBAL`/`SET PERSIST`). Statique : modification dans un fichier `.cnf` ou `SET PERSIST_ONLY`, effective au prochain redémarrage seulement. |
| **Slow query log** | Journal des requêtes dépassant `long_query_time`, désactivé par défaut — premier outil de diagnostic de performance à activer en production. |
| **`sync_binlog` / `innodb_flush_log_at_trx_commit`** | Couple de paramètres contrôlant la durabilité : tous deux à `1` garantit zéro perte de données en cas de crash, au prix d'écritures disque doublées par commit. |
| **`bind_address`** | Adresse(s) réseau sur lesquelles `mysqld` écoute — variable statique, un restart est nécessaire pour la modifier. |
| **`innodb_dedicated_server`** | Mode auto-dimensionnant `innodb_buffer_pool_size` et `innodb_redo_log_capacity` selon la RAM/CPU détectés, réservé à un serveur 100 % dédié à MySQL. |
| **Sauvegarde logique** | Export sous forme d'instructions SQL (`mysqldump`) ou de fichiers compressés (`mysqlsh`), portable entre versions mais plus lent qu'une sauvegarde physique. |
| **Sauvegarde physique** | Copie brute des fichiers de données InnoDB (XtraBackup), rapide mais liée à une version majeure précise de MySQL. |
| **`--single-transaction`** | Option de `mysqldump` démarrant une transaction `REPEATABLE READ` pour un dump cohérent sans verrouiller les tables InnoDB. |
| **Percona XtraBackup** | Outil open source de sauvegarde physique à chaud des fichiers InnoDB, en trois étapes : backup, prepare, restore. |
| **RPO (Recovery Point Objective)** | Volume de données qu'une organisation accepte de perdre lors d'un incident — détermine la fréquence de sauvegarde et d'archivage des binlogs nécessaire. |
| **RTO (Recovery Time Objective)** | Durée d'indisponibilité acceptable lors d'une restauration — détermine le choix entre sauvegarde logique et physique. |
| **PITR (Point-In-Time Recovery)** | Restauration à un instant précis, combinant un dump de référence et le rejeu des binary logs jusqu'à ce point. |
| **`mysqlbinlog`** | Outil en ligne de commande lisant et filtrant le contenu des binary logs (par date, position ou GTID). |
| **`FLUSH BINARY LOGS`** | Force la rotation du binlog courant, indispensable avant de copier les fichiers vers un stockage externe. |
| **`SHOW PROCESSLIST`** | Commande listant les connexions actives et leur état — tronque la colonne `Info` à 100 caractères, contrairement à `performance_schema.processlist`. |
| **Connexion Sleep** | Connexion ouverte mais inactive (`Command = Sleep`) — leur accumulation signale un pool applicatif mal fermé ou un `wait_timeout` trop élevé. |
| **Buffer pool hit ratio** | Proportion de lectures servies depuis le buffer pool plutôt que depuis le disque — indicateur de performance InnoDB le plus surveillé (objectif > 99% en OLTP). |
| **`sys.statement_analysis`** | Vue `sys` listant les requêtes par coût total/moyen, équivalent MySQL de `pg_stat_statements` de PostgreSQL. |
| **`EXPLAIN` / `EXPLAIN ANALYZE`** | Commandes affichant le plan d'exécution prévu (`EXPLAIN`) ou réellement mesuré (`EXPLAIN ANALYZE`) d'une requête. |
| **Type d'accès (EXPLAIN)** | Colonne `type` d'`EXPLAIN` indiquant la méthode de lecture utilisée, du pire (`ALL`, full table scan) au meilleur (`system`/`const`). |
| **History list length** | Nombre de transactions en attente de purge par InnoDB (MVCC) — une valeur croissante signale un thread de purge dépassé par le rythme des écritures. |
| **Adaptive hash index (AHI)** | Mécanisme automatique d'InnoDB construisant un index hash en mémoire pour accélérer les lookups fréquents — désactivé par défaut depuis MySQL 8.4. |
| **`ANALYZE TABLE`** | Commande rafraîchissant les statistiques utilisées par l'optimiseur de requêtes, à exécuter après tout chargement massif. |
| **Histogramme (MySQL)** | Statistique décrivant la distribution des valeurs d'une colonne, au-delà de sa simple cardinalité — utile pour les colonnes à distribution non uniforme. |
| **`OPTIMIZE TABLE`** | Commande reconstruisant une table InnoDB pour récupérer l'espace libéré par des `DELETE`/`UPDATE`, via une copie temporaire complète. |
| **`CHECK TABLE` / `mysqlcheck` / `innochecksum`** | Outils de vérification d'intégrité — `CHECK TABLE` peut bloquer des threads sur une grosse table InnoDB, `innochecksum` vérifie un fichier `.ibd` hors ligne sans ce risque. |
| **`mysqld_exporter`** | Exportateur Prometheus exposant les métriques MySQL (`SHOW GLOBAL STATUS`, InnoDB, réplication) pour une supervision continue. |
| **`authentication_policy`** | Variable contrôlant les exigences d'authentification multifacteur par facteur, remplaçant `default_authentication_plugin` (supprimée en MySQL 8.4). |
| **`validate_password`** | Composant imposant une politique de complexité des mots de passe (`LOW`/`MEDIUM`/`STRONG`), installé par défaut par `mysql_secure_installation`. |
| **`CREATE ROLE`** | Crée un rôle — un ensemble nommé de privilèges attribuable à plusieurs comptes utilisateurs via `GRANT role TO user`. |
| **`SET DEFAULT ROLE`** | Active automatiquement un rôle à la connexion — sans cela, un rôle accordé (`GRANT`) reste inactif tant qu'il n'est pas activé manuellement (`SET ROLE`) ou globalement (`activate_all_roles_on_login`). |
| **Privilèges granulaires (8.4)** | Décomposition de l'ancien privilège monolithique `SUPER` en droits spécifiques (`FLUSH_PRIVILEGES`, `SYSTEM_VARIABLES_ADMIN`, `CONNECTION_ADMIN`, `ROLE_ADMIN`...), permettant d'appliquer le moindre privilège. |
| **`partial_revokes`** | Fonctionnalité (8.0.16+) permettant de révoquer un privilège global sur un schéma précis — utile pour protéger le schéma `mysql` sans réduire les autres droits d'un compte. |
| **`REQUIRE SSL` / `REQUIRE X509`** | Clauses de `CREATE USER`/`ALTER USER` forçant le chiffrement TLS pour un compte (`SSL`), avec vérification du certificat client (`X509`). |
| **`--ssl-mode`** | Paramètre client contrôlant le niveau de vérification TLS (`DISABLED`, `PREFERRED`, `REQUIRED`, `VERIFY_CA`, `VERIFY_IDENTITY`) — distinct du `REQUIRE` côté serveur. |
| **`connection_control`** | Plugin ralentissant les tentatives de connexion après un nombre d'échecs configuré, protection basique contre le brute-force. |
| **`server_id`** | Identifiant numérique unique et non nul de chaque serveur d'une topologie de réplication — deux serveurs partageant le même ID cassent la réplication. |
| **Source / Replica** | Terminologie MySQL 8.4 (remplace *master*/*slave*) : le source écrit dans le binary log, le replica lit et applique ces événements. |
| **GTID set** | Ensemble de tous les GTID exécutés sur un serveur, visible dans `gtid_executed` — permet au replica de savoir exactement quelles transactions il a déjà appliquées. |
| **Clone plugin** | Plugin MySQL 8.0+ copiant l'intégralité du data directory d'un donor vers un recipient — équivalent MySQL de `pg_basebackup -R` de PostgreSQL. |
| **`REPLICATION SLAVE`** | Privilège n'autorisant que la lecture du binary log d'un serveur, sans accès aux données — à réserver à un compte dédié à la réplication. |
| **`CHANGE REPLICATION SOURCE TO`** | Commande (MySQL 8.0.23+, remplace `CHANGE MASTER TO`) configurant la connexion d'un replica à son source. |
| **`SOURCE_AUTO_POSITION`** | Option de `CHANGE REPLICATION SOURCE TO` activant le repositionnement automatique par GTID, sans calculer manuellement fichier/offset binlog. |
| **`Seconds_Behind_Source`** | Colonne de `SHOW REPLICA STATUS` indiquant le lag en secondes entre source et replica — métrique utile mais approximative. |
| **`read_only` / `super_read_only`** | Paramètres bloquant les écritures sur un replica — `super_read_only` bloque aussi les comptes `SUPER`/`CONNECTION_ADMIN`, à activer systématiquement ensemble. |
| **Failover** | Promotion d'un replica en nouveau source suite à l'indisponibilité de l'ancien — manuel par défaut en MySQL, automatique avec Group Replication. |
| **Réplication semi-synchrone** | Mode où le `COMMIT` du source attend la confirmation d'au moins un replica avant de répondre au client — RPO quasi nul, au prix d'une latence réseau. |
| **Group Replication (GR)** | Solution HA intégrée à MySQL basée sur un consensus distribué (variante de Paxos) entre 3 nœuds minimum, avec détection de pannes et failover automatiques. |
| **InnoDB Cluster** | Combinaison de Group Replication + MySQL Shell + MySQL Router, la solution haute disponibilité complète d'Oracle. |
| **InnoDB ReplicaSet** | Orchestration simplifiée (via MySQL Shell) de la réplication asynchrone classique, alternative moins contraignante à InnoDB Cluster. |
| **MySQL Router** | Proxy officiel Oracle routant les connexions vers le primary (écritures) ou les secondaries (lectures) d'un InnoDB Cluster. |
