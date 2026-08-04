#bdd #mariadb #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **MariaDB Foundation** | Organisation à but non lucratif garante du code ouvert et de la gouvernance communautaire de MariaDB. |
| **MariaDB Corporation** | Entreprise commerciale portant les éditions Enterprise et le support payant de MariaDB. |
| **`mariadbd`** | Nom du processus serveur MariaDB — équivalent de `mysqld` côté MySQL. |
| **`mariadb` (client)** | Client en ligne de commande officiel depuis MariaDB 10.4+ — le nom `mysql` reste disponible en alias pour compatibilité. |
| **`mariadb-secure-installation`** | Script post-installation supprimant comptes anonymes et base `test`, équivalent de `mysql_secure_installation`. |
| **`mysql.global_priv`** | Table (depuis MariaDB 10.4) stockant comptes et privilèges globaux en JSON — `mysql.user` n'est plus qu'une vue au-dessus. |
| **Aria** | Moteur de stockage MariaDB, successeur crash-safe de MyISAM, utilisé par défaut pour toutes les tables système depuis la version 10.4. |
| **`TRANSACTIONAL` (Aria)** | Option activant un journal de transactions propre à Aria, protégeant contre la perte de données en cas de crash serveur. |
| **ColumnStore** | Moteur de stockage MariaDB orienté colonnes, pour les charges analytiques (OLAP/HTAP) sur de gros volumes. |
| **Spider** | Moteur de stockage virtuel MariaDB redirigeant les requêtes vers des tables distantes, pour le sharding de données. |
| **MyRocks** | Moteur de stockage basé sur RocksDB (LSM-tree), pour les charges d'écriture intensive avec forte compression. |
| **Sequence** | Objet SQL autonome générant des valeurs via `NEXT VALUE FOR`, alternative à `AUTO_INCREMENT` non liée à une seule table. |
| **Table System-Versioned** | Table (`WITH SYSTEM VERSIONING`) conservant automatiquement l'historique de ses lignes modifiées, interrogeable via `FOR SYSTEM_TIME`. |
| **`ROW_START` / `ROW_END`** | Colonnes internes ajoutées par MariaDB à une table System-Versioned, délimitant la période de validité de chaque version de ligne. |
| **LTS (MariaDB)** | Série à support long, publiée en coordination avec les cycles des distributions Linux — actuellement la version `.3` de chaque série majeure. |
| **`unix_socket`** | Plugin d'authentification par défaut de `root@localhost`, basé sur l'identité de l'utilisateur du système d'exploitation. |
| **`ed25519`** | Plugin d'authentification MariaDB basé sur la signature elliptique EdDSA, recommandé pour les comptes réseau (installation manuelle requise). |
| **`IDENTIFIED VIA ... USING`** | Syntaxe MariaDB idiomatique pour associer un plugin d'authentification à un compte, alternative à `IDENTIFIED WITH ... BY` de MySQL. |
| **`mariadb-dump`** | Outil de sauvegarde logique de MariaDB, équivalent de `mysqldump`. |
| **`mariadb-backup`** | Outil de sauvegarde physique officiel de MariaDB, équivalent de Percona XtraBackup. |
| **GTID MariaDB (`domaine-serveur-séquence`)** | Format de transaction globale propre à MariaDB (ex. `0-1-345`), incompatible avec le format `UUID:transaction_id` de MySQL. |
| **`gtid_domain_id`** | Identifiant de flux de réplication dans le GTID MariaDB, permettant nativement la réplication multi-source. |
| **`gtid_current_pos`** | Variable système donnant la position GTID courante d'un nœud — union de `gtid_binlog_pos` et `gtid_slave_pos`. |
| **Galera Cluster** | Solution de réplication synchrone multi-maître intégrée à MariaDB, basée sur la certification de write-sets via l'API wsrep. |
| **Write-set (Galera)** | Ensemble des modifications d'une transaction, diffusé à tous les nœuds du cluster Galera au moment du `COMMIT`, avant certification. |
| **Certification (Galera)** | Vérification, sur chaque nœud, qu'un write-set reçu n'entre pas en conflit avec une transaction concurrente déjà appliquée. |
| **Quorum (Galera)** | Majorité de nœuds nécessaire pour qu'un cluster Galera reste `Primary` (accepte les écritures) après une partition réseau. |
| **`wsrep_cluster_status`** | Variable de statut Galera : `Primary` (quorum atteint) ou `Non-Primary` (quorum perdu, écritures refusées). |
| **Streaming replication (Galera)** | Mécanisme (Galera 4+) fragmentant une transaction volumineuse en plusieurs write-sets certifiés progressivement. |
