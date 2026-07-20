#bdd #mysql #innodb #avancé

## Le redo log : Write-Ahead Logging

Le **redo log** est le journal de crash recovery d'InnoDB. Chaque modification de données est écrite dans le redo log **avant** d'être appliquée aux fichiers de données — le mécanisme *Write-Ahead Logging*, le même principe que les WAL de PostgreSQL.

> [!tip] Analogie : le carnet de bord
> Avant de ranger un livre dans la bibliothèque (fichier de données), on note dans le carnet « ranger le livre X à l'emplacement Y ». Si la bibliothèque est renversée (crash serveur), tout se reconstitue en relisant le carnet.

```bash
ls /var/lib/mysql/#innodb_redo/
# #ib_redo10  #ib_redo11  #ib_redo12_tmp  ...
```

Les fichiers sans `_tmp` sont actifs (en cours d'écriture) ; ceux avec `_tmp` sont de réserve, recyclés quand les actifs sont pleins. Depuis MySQL 8.0.30, la capacité globale se pilote via `innodb_redo_log_capacity` (les anciennes variables `innodb_log_file_size`/`innodb_log_files_in_group` sont dépréciées).

> [!info] Redo log ≠ binary log
> Le redo log et les WAL PostgreSQL servent le même objectif (durabilité, crash recovery). Mais la réplication MySQL ne repose **pas** sur le redo log — elle repose sur le binary log, un journal distinct. Voir [[MySQL 07 — Binary log — réplication & PITR]].

## Le doublewrite buffer : protéger contre les écritures partielles

Une page InnoDB fait 16 Ko, mais le système de fichiers peut écrire par blocs de 4 Ko — un crash pendant l'écriture d'une page ne laisserait qu'une partie écrite, corrompant la page.

Mécanisme : avant d'écrire une page modifiée à son emplacement final, InnoDB l'écrit d'abord dans le **doublewrite buffer** (des fichiers dédiés). Si un crash interrompt l'écriture finale, InnoDB retrouve la copie intacte dans le doublewrite buffer au redémarrage et la réécrit.

```bash
ls /var/lib/mysql/#ib_*dblwr*
# #ib_16384_0.dblwr  #ib_16384_1.dblwr
```

> [!warning] Ne jamais désactiver le doublewrite buffer sur ext4/XFS
> Seuls des systèmes de fichiers garantissant des écritures atomiques de 16 Ko (ZFS, certains SAN) permettraient de s'en passer sans risque. Sur ext4 ou XFS, le risque de corruption est réel sans cette protection.

## Tablespaces : système vs fichier-par-table

| Type | Contenu | Fichier |
|------|---------|---------|
| Tablespace système | Change buffer et structures internes | `ibdata1` |
| Tablespace fichier-par-table | Données + index d'**une seule table** | `<table>.ibd`, un par table (défaut depuis MySQL 5.6) |

```bash
ls /var/lib/mysql/labdb/
# servers.ibd  deployments.ibd  events.ibd  ...
```

| Avantage du fichier-par-table | Explication |
|-----------------------------------|-------------|
| Gestion de l'espace | Un `DROP TABLE` libère immédiatement l'espace disque |
| Sauvegarde ciblée | Copier/restaurer une seule table devient possible |
| Compression | Chaque table peut être compressée indépendamment |
| Monitoring | La taille de chaque table est visible directement dans le filesystem |

## Le change buffer : optimiser les écritures d'index

Quand une ligne est insérée, MySQL doit aussi mettre à jour ses index secondaires. Si la page d'index correspondante n'est pas déjà en mémoire, la charger depuis le disque serait coûteux — le **change buffer** met en cache cette modification, appliquée plus tard quand la page sera de toute façon lue (par un `SELECT` ou un merge ultérieur). Réduit considérablement les I/O aléatoires sur des charges à forte écriture.

## Cas particuliers

> [!warning] Ne jamais copier `auto.cnf` sur un réplica
> Ce fichier contient le `server_uuid`, identifiant unique de l'instance. Cloner un serveur (snapshot VM, copie de `/var/lib/mysql/`) sans supprimer `auto.cnf` sur le clone avant démarrage laisse deux serveurs avec le même UUID — source d'erreurs de réplication. MySQL en génère un nouveau automatiquement si le fichier est absent.

## Pour aller plus loin

Le journal de réplication (binary log), distinct du redo log, est détaillé dans [[MySQL 07 — Binary log — réplication & PITR]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
