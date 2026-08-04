#bdd #home #index

## Modules disponibles

### Fondamentaux

- [[BDD — Généralités]] ← historique, concepts clés (schéma, index, transactions, ACID vs BASE, partitionnement, sharding, cache)
- [[BDD — Types de bases de données]] ← relationnel, NoSQL (document/clé-valeur/colonnes/graphe), mémoire, distribué
- [[BDD — Fonctionnalités des serveurs]] ← transactions, contrôle d'accès, sauvegarde, réplication, index, monitoring, sécurité

### Référence

- [[BDD — Glossaire]]

### Langage de requête

- [[SQL — Index des fiches]]

### Moteurs relationnels

- [[MySQL — Index des fiches]] ← architecture, InnoDB, réplication
- [[MariaDB — Index des fiches]] ← fork MySQL, Aria, Sequences & System Versioning, Galera Cluster
- [[SQLite — Index des fiches]] ← base embarquée mono-fichier, WAL, sans serveur ni réseau

### Moteurs clé-valeur / mémoire

- [[Redis — Index des fiches]] ← licence & fork Valkey, types de données, persistance RDB/AOF, réplication, Sentinel, Cluster

### Outils d'administration

- [[Adminer — Index des fiches]] ← client web mono-fichier, multi-moteurs (MySQL, PostgreSQL, SQLite...), sécurisation & déploiement

## Parcours recommandés

```
Fondamentaux : BDD — Généralités → Types de bases de données → Fonctionnalités des serveurs
Relationnel  : SQL 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12
MySQL        : MySQL 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13 → 14 → 15 → 16 → 17 → 18 → 19 → 20 → 21 → 22 → 23 → 24 → 25 → 26 → 27 → 28 → 29 → 30
MariaDB      : MariaDB 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13
SQLite       : SQLite 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07
Redis        : Redis 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13 → 14 → 15
Adminer      : Adminer 00 → 01 → 02 → 03 → 04 → 05 → 06
```

## Prérequis & suite

- [[Home]] ← retour accueil
- [[Pandas — Index des fiches]] ← suite logique côté SQL (opérations similaires : groupby, merge, agg)
- [[Manques]] ← SQLAlchemy (ORM Python sur SQL, non couvert), PostgreSQL (alternative de migration à SQLite, non couvert), MongoDB (non couvert), phpMyAdmin & pgAdmin (comparés à Adminer mais non couverts en détail)
