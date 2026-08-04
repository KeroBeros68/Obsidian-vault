#bdd #mariadb #pièges #erreurs #debugging

## 🪤 Piège 1 — Éditer mysql.global_priv directement

```sql
-- ❌ Modification directe de la table interne
UPDATE mysql.global_priv SET Priv = JSON_SET(Priv, '$.plugin', 'ed25519') WHERE User = 'alice';
```

> [!warning] Passer par les commandes SQL standard
> Depuis MariaDB 10.4, `mysql.user` n'est qu'une vue au-dessus de `mysql.global_priv`, qui stocke les privilèges en JSON. Modifier cette table à la main peut corrompre le format interne — utiliser `CREATE USER`/`ALTER USER`/`GRANT`/`REVOKE` exclusivement. Voir [[MariaDB 02 — Bases de données & bases système]].

---

## 🪤 Piège 2 — Espérer répliquer un GTID MySQL vers un replica MariaDB

```sql
CHANGE MASTER TO MASTER_USE_GTID = slave_pos;
-- Échoue si le source est un MySQL : formats GTID incompatibles
```

> [!warning] Les deux formats GTID sont étanches
> Le GTID MariaDB (`domaine-serveur-séquence`) et le GTID MySQL (`UUID:transaction_id`) ne s'interopèrent pas. Une réplication croisée MariaDB ↔ MySQL doit revenir à la position de binlog classique (fichier + offset). Voir [[MariaDB 11 — Réplication classique & GTID MariaDB]].

---

## 🪤 Piège 3 — Utiliser SET GLOBAL en pensant que ça persiste

```sql
SET GLOBAL innodb_buffer_pool_size = 4294967296;
-- Perdu au prochain redémarrage : MariaDB n'a pas de SET PERSIST
```

> [!warning] Pas d'équivalent à SET PERSIST
> Contrairement à MySQL 8.0+, MariaDB ne propose aucun mécanisme pour rendre un `SET GLOBAL` permanent. Le changement doit être reporté manuellement dans le fichier `.cnf` pour survivre à un redémarrage. Voir [[MariaDB 09 — Configuration (my.cnf, InnoDB & Aria)]].

---

## 🪤 Piège 4 — Créer une table Galera sans clé primaire

```sql
-- ❌ Sur un cluster Galera
CREATE TABLE logs (message TEXT) ENGINE=InnoDB;
```

> [!warning] Galera exige une clé primaire sur chaque table
> Sans clé primaire, la certification des write-sets ne peut pas fonctionner correctement — les lignes ne sont pas identifiables de manière unique pour détecter les conflits entre nœuds. Toujours définir un `PRIMARY KEY` sur les tables répliquées par Galera. Voir [[MariaDB 12 — Galera Cluster, réplication synchrone multi-maître]].

---

## 🪤 Piège 5 — Déployer un cluster Galera à deux nœuds

> [!warning] Un cluster à 2 nœuds n'a pas de quorum fiable
> Avec seulement 2 nœuds, une coupure réseau isole chaque nœud à égalité (1 contre 1) — aucun des deux ne peut déterminer s'il détient la majorité. Toujours déployer Galera avec 3 nœuds minimum (nombre impair recommandé), ou ajouter un *arbitrator* (`garbd`) si un troisième nœud complet n'est pas justifié. Voir [[MariaDB 12 — Galera Cluster, réplication synchrone multi-maître]].

---

## 🪤 Piège 6 — Oublier d'installer ed25519 avant de créer un compte

```sql
CREATE USER 'app'@'10.0.1.%' IDENTIFIED VIA ed25519 USING PASSWORD('...');
-- ERROR 1524 (HY000): Plugin 'ed25519' is not loaded
```

> [!tip] INSTALL SONAME d'abord
> Contrairement à `unix_socket` (actif par défaut), le plugin `ed25519` est distribué avec MariaDB mais pas installé automatiquement — exécuter `INSTALL SONAME 'auth_ed25519';` avant de créer le premier compte qui l'utilise. Voir [[MariaDB 08 — Authentification (unix_socket, ed25519 & mysql_native_password)]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Édition directe de `mysql.global_priv` | Passer par `CREATE USER`/`GRANT`/`REVOKE` |
| Réplication GTID croisée MariaDB ↔ MySQL | Revenir à la position de binlog classique |
| `SET GLOBAL` supposé persistant | Reporter le changement dans le fichier `.cnf` |
| Table Galera sans clé primaire | Toujours définir un `PRIMARY KEY` |
| Cluster Galera à 2 nœuds | 3 nœuds minimum, ou `garbd` en arbitre |
| Compte `ed25519` créé sans installer le plugin | `INSTALL SONAME 'auth_ed25519';` d'abord |
