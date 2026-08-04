#bdd #mariadb #fondamentaux

## Une architecture pluggable, héritée puis étendue

Comme MySQL (voir [[MySQL 04 — Moteurs de stockage (InnoDB vs les autres)]]), MariaDB sépare la couche SQL de la couche de stockage : chaque table déclare son moteur (`ENGINE=...`), et différentes tables d'une même base peuvent utiliser des moteurs différents. MariaDB a hérité de cette architecture de MySQL, puis l'a considérablement enrichie avec des moteurs que MySQL ne propose pas.

```sql
SHOW ENGINES;
SELECT @@default_storage_engine;
-- InnoDB
```

## Panorama des moteurs disponibles

| Moteur | Usage principal | Spécifique à MariaDB |
|--------|-------------------|------------------------|
| **InnoDB** | Transactionnel généraliste, moteur par défaut pour les tables utilisateur | Non (partagé avec MySQL, voir [[MariaDB 05 — InnoDB dans MariaDB]]) |
| **Aria** | Tables système, tables temporaires internes, lecture intensive | Oui |
| **MyISAM** | Legacy, lecture intensive sans transactions | Non (hérité de MySQL, en déclin) |
| **Memory** | Données temporaires en RAM, cache | Non |
| **ColumnStore** | Analytique, OLAP/HTAP sur gros volumes (architecture distribuée) | Oui |
| **Spider** | Sharding : partitionnement des données sur plusieurs serveurs | Oui |
| **MyRocks** | Écriture intensive, forte compression (basé sur RocksDB, LSM-tree) | Oui |
| **S3** | Stockage en lecture seule directement sur un bucket S3 | Oui |

> [!info] InnoDB reste le choix par défaut
> Malgré la richesse de ce catalogue, `InnoDB` reste le moteur par défaut pour les tables créées sans `ENGINE` explicite — les mêmes principes ACID et MVCC que ceux couverts pour MySQL s'appliquent (voir [[MySQL 05 — InnoDB — le buffer pool]]).

## Pourquoi tant de moteurs différents

Chaque moteur cible un compromis différent :

- **Aria** vise à remplacer MyISAM avec de la sécurité en cas de crash, sans le coût transactionnel complet d'InnoDB.
- **MyRocks** cible les workloads d'écriture intensive sur SSD, avec une compression bien supérieure à InnoDB grâce à sa structure LSM-tree (*Log-Structured Merge-tree*), héritée du projet RocksDB de Facebook/Meta.
- **ColumnStore** stocke les données en colonnes plutôt qu'en lignes, optimisé pour les requêtes analytiques (`SUM`, `AVG`, `GROUP BY` sur de larges volumes) plutôt que pour les transactions.
- **Spider** ne stocke rien lui-même : c'est un moteur *virtuel* qui redirige les requêtes vers des tables distantes, permettant de distribuer (*sharder*) une même base sur plusieurs instances MariaDB.

> [!warning] Un moteur non standard change les garanties disponibles
> Passer une table sur MyRocks, ColumnStore ou Spider change fondamentalement ses garanties transactionnelles et ses performances par opération. Ces moteurs répondent à des besoins spécifiques (compression, analytique, sharding) — ils ne sont pas des remplacements généralistes d'InnoDB.

## Pour aller plus loin

Le moteur Aria, utilisé par toutes les tables système MariaDB, est détaillé dans [[MariaDB 04 — Aria, le moteur des tables système]].

Sources : [Storage Engines Overview — MariaDB Documentation](https://mariadb.com/docs/server/server-usage/storage-engines/storage-engines-storage-engines-overview), [MariaDB and MySQL storage engines: an overview — Vettabase](https://vettabase.com/mariadb-and-mysql-storage-engines-an-overview/)
