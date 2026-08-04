#bdd #sqlite #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Base embarquée** | Moteur de base de données fonctionnant comme une bibliothèque intégrée à l'application, sans processus serveur ni port réseau. |
| **`sqlite3` (CLI)** | Outil en ligne de commande pour créer, requêter, inspecter et sauvegarder une base SQLite — distinct de la bibliothèque SQLite elle-même. |
| **Commande dot** | Commande d'administration de la CLI `sqlite3` commençant par `.` (`.tables`, `.schema`, `.mode`, `.dump`, `.backup`), non exécutée par le moteur SQL. |
| **`SQLITE_BUSY`** | Erreur retournée à un processus qui tente d'écrire pendant qu'un autre détient déjà le verrou d'écriture. |
| **Journal rollback** | Mécanisme de journalisation par défaut : copie des pages modifiées dans un fichier `.db-journal` avant écriture, pour permettre un retour en arrière en cas de crash — bloque aussi les lectures pendant une écriture. |
| **WAL (Write-Ahead Logging)** | Mode de journalisation alternatif écrivant les modifications dans un fichier séparé (`.db-wal`), permettant aux lecteurs de continuer pendant une écriture. |
| **`.db-wal` / `.db-shm`** | Fichiers persistants produits par le mode WAL — à toujours copier avec le fichier `.db` principal, jamais à supprimer manuellement. |
| **STRICT table** | Table déclarée avec `STRICT` (depuis la version 3.37) imposant un typage réel des colonnes, contrairement au typage dynamique permissif par défaut de SQLite. |
| **Colonne générée** | Colonne dont la valeur est calculée automatiquement à partir d'autres colonnes (`GENERATED ALWAYS AS ... STORED`), depuis la version 3.31. |
| **`json_extract()`** | Fonction SQL extrayant une valeur d'un champ JSON stocké en texte, via un chemin de type `$.champ`. |
| **`.backup`** | Commande de la CLI produisant une copie cohérente de la base, même avec des lectures en cours. |
| **`VACUUM INTO`** | Instruction SQL créant une copie compactée de la base dans un nouveau fichier, en éliminant la fragmentation. |
| **`sqlite3_rsync`** | Outil (depuis la version 3.50) synchronisant une base vers un serveur distant en ne transmettant que les pages modifiées. |
| **`PRAGMA integrity_check`** | Commande parcourant l'ensemble des pages et index pour détecter une corruption, renvoyant `ok` ou la liste des anomalies. |
| **`PRAGMA optimize`** | Met à jour les statistiques utilisées par l'optimiseur de requêtes — à exécuter avant de fermer une connexion de longue durée. |
| **SQLCipher** | Variante de SQLite ajoutant un chiffrement AES-256 transparent du fichier de base, protégé par une passphrase. |
| **RDBMS embarqué vs client/serveur** | Distinction structurante : SQLite (embarqué, mono-hôte, un seul écrivain) contre MySQL/PostgreSQL (serveur réseau, rôles, réplication, haute disponibilité). |
