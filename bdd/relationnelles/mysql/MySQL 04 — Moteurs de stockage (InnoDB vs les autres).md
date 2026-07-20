#bdd #mysql #innodb #intermédiaire

## InnoDB, le moteur par défaut depuis 2010

**InnoDB** est le moteur de stockage par défaut de MySQL depuis la version 5.5 (2010) — le seul qui offre l'ensemble des garanties attendues d'une base de données moderne.

| Caractéristique | InnoDB | MyISAM |
|--------------------|--------|--------|
| Transactions ACID | ✅ `COMMIT`/`ROLLBACK` | ❌ |
| Clés étrangères | ✅ `FOREIGN KEY` | ❌ |
| Verrouillage | ✅ Ligne (row-level) | ❌ Table (table-level) |
| Crash recovery | ✅ Via redo log | ❌ Réparation manuelle |
| MVCC | ✅ Lectures non bloquantes | ❌ |

**ACID** : Atomicité (une transaction réussit ou échoue entièrement), Cohérence (les contraintes sont toujours respectées), Isolation (les transactions concurrentes ne se voient pas), Durabilité (une transaction commitée survit à un crash).

**MVCC** (*Multi-Version Concurrency Control*) : les lectures ne bloquent pas les écritures, et inversement — chaque transaction voit un snapshot cohérent des données, même si d'autres transactions modifient les mêmes lignes au même moment. C'est ce qui permet à MySQL de gérer des milliers de connexions simultanées sans que les `SELECT` ne soient bloqués par les `UPDATE`.

```sql
SHOW TABLE STATUS FROM labdb WHERE Name = 'servers' \G
-- Engine: InnoDB
-- Row_format: Dynamic
```

## Les autres moteurs : des cas d'usage très spécifiques

| Moteur | Usage | Limites |
|--------|-------|---------|
| Memory (HEAP) | Tables temporaires en RAM, caches de session | Données perdues au redémarrage, pas de transactions |
| MyISAM | Legacy uniquement | Pas de transactions, verrouillage table, crash = réparation manuelle |
| NDB (Cluster) | Haute disponibilité distribuée (télécom, temps réel) | Architecture complexe |
| ARCHIVE | Logs historiques en lecture seule compressés | Pas d'index, pas d'`UPDATE`/`DELETE` |
| CSV | Import/export de fichiers CSV | Pas d'index, pas de transactions |

```sql
SHOW ENGINES;
-- InnoDB est DEFAULT, supporte transactions/row-locking/foreign keys
```

> [!warning] MyISAM est obsolète — migrer toute table héritée
> MyISAM ne supporte ni les transactions, ni les clés étrangères, ni le crash recovery : une coupure de courant pendant une écriture peut corrompre les données de façon irrécupérable. Migration simple :
> ```sql
> ALTER TABLE ma_table ENGINE=InnoDB;
> ```

## Cas particuliers

> [!info] Un choix de moteur par table, pas par instance
> Le moteur de stockage se définit par table (`ENGINE=InnoDB`), pas globalement pour toute l'instance — une base peut techniquement mélanger des tables InnoDB et MyISAM, même si ce n'est recommandé pour aucune donnée de production.

## Pour aller plus loin

L'architecture interne d'InnoDB (buffer pool, redo log, tablespaces) est détaillée dans [[MySQL 05 — InnoDB — le buffer pool]] et [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
