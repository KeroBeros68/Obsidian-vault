#bdd #mariadb #avancé

## Deux outils, deux philosophies

MariaDB fournit deux outils de sauvegarde officiels, sur le même principe que MySQL (`mysqldump` vs XtraBackup, voir [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]]) : **`mariadb-dump`** pour une sauvegarde logique, et **`mariadb-backup`** pour une sauvegarde physique.

| | `mariadb-dump` | `mariadb-backup` |
|---|----------------|---------------------|
| Type | Logique (instructions SQL) | Physique (copie de fichiers) |
| Contenu | `CREATE TABLE`, `INSERT`... | Fichiers InnoDB/Aria/MyISAM bruts |
| Vitesse de sauvegarde | Plus lente (un thread par défaut) | Rapide (parallélisée par cœur CPU) |
| Vitesse de restauration | Lente (rejoue chaque `INSERT`, reconstruit les index) | Rapide (copie de fichiers, pas de rejeu SQL) |
| Portabilité | Vers une autre version, un autre matériel, voire un autre SGBD | Liée à une version proche de MariaDB et à un matériel compatible |
| Cas d'usage | Petits volumes, portabilité, migration | Gros volumes, RTO court |

> [!warning] La restauration logique est le vrai coût caché
> Sur une base avec de nombreux index secondaires, reconstruire les index après un `mariadb-dump` peut représenter jusqu'à 70 % du temps total de restauration. `mariadb-backup` évite ce coût : la restauration se résume à recopier les fichiers et démarrer MariaDB, sans rejeu SQL ni reconstruction d'index.

## mariadb-dump : sauvegarde logique

```bash
mariadb-dump -u root -p --single-transaction --routines --triggers --events \
  --all-databases > /tmp/dump_complet.sql

mariadb -u root -p < /tmp/dump_complet.sql
```

`--single-transaction` évite de verrouiller les tables InnoDB pendant le dump, exactement comme pour `mysqldump`.

## mariadb-backup : sauvegarde physique

```bash
# Sauvegarde
mariadb-backup --backup --target-dir=/backup/full --user=root --password=...

# Préparation (applique les transactions en cours au moment du backup)
mariadb-backup --prepare --target-dir=/backup/full

# Restauration (serveur arrêté, datadir vide)
mariadb-backup --copy-back --target-dir=/backup/full --datadir=/var/lib/mysql
chown -R mysql:mysql /var/lib/mysql
```

> [!info] BACKUP STAGE : un verrouillage minimal
> `mariadb-backup` s'appuie sur les commandes `BACKUP STAGE` et la journalisation des DDL pour minimiser les verrous pendant la copie — notamment en évitant tout verrou pendant la phase de copie des `ALTER TABLE`, habituellement l'étape la plus longue de ces instructions.

## Choisir entre les deux

- **`mariadb-dump`** : petites bases, besoin de portabilité entre versions ou entre SGBD, migration ponctuelle.
- **`mariadb-backup`** : grosses bases en production, contrainte de RTO (*Recovery Time Objective*) courte, sauvegarde à chaud fréquente.

Les deux outils peuvent coexister dans une même stratégie de sauvegarde : `mariadb-backup` pour les sauvegardes complètes régulières, `mariadb-dump` pour des exports ponctuels ciblés (une seule base, avant une migration de schéma).

## Pour aller plus loin

La réplication classique par GTID, dont le format diffère de celui de MySQL, est détaillée dans [[MariaDB 11 — Réplication classique & GTID MariaDB]].

Sources : [mariadb-backup Overview — MariaDB Documentation](https://mariadb.com/docs/server/server-usage/backup-and-restore/mariadb-backup/mariadb-backup-overview), [mariadb-dump — MariaDB Documentation](https://mariadb.com/docs/server/clients-and-utilities/backup-restore-and-import-clients/mariadb-dump), [Full Backup and Restore (mariadb-backup) — MariaDB Documentation](https://mariadb.com/docs/server/server-usage/backup-and-restore/mariadb-backup/full-backup-and-restore-with-mariadb-backup)
