#bdd #mysql #maintenance #avancé

## ANALYZE TABLE : rafraîchir les statistiques de l'optimiseur

L'optimiseur choisit son plan d'exécution (voir [[MySQL 17 — EXPLAIN & EXPLAIN ANALYZE]]) à partir de statistiques sur la distribution des données. Après un gros chargement ou des suppressions massives, des statistiques obsolètes mènent à de mauvais choix de plan.

```sql
ANALYZE TABLE lab_mysql.logs;
```

> [!tip] À exécuter après tout chargement massif
> `ANALYZE TABLE` est généralement rapide sur InnoDB, mais prend un *read lock* pendant l'opération — à planifier hors pic si la table est très sollicitée en écriture.

> [!info] Statistiques persistantes par défaut depuis MySQL 8.0
> `innodb_stats_persistent = ON` par défaut : les statistiques sont stockées sur disque et survivent au redémarrage. Avant MySQL 8.0, elles étaient recalculées à chaque ouverture de table.

## Histogrammes : une vision fine de la distribution

Les histogrammes donnent à l'optimiseur une vision détaillée de la distribution des valeurs d'une colonne (pas seulement sa cardinalité) — particulièrement utile pour des colonnes à distribution non uniforme.

```sql
ANALYZE TABLE lab_mysql.logs UPDATE HISTOGRAM ON action, client_id;
```

```sql
-- Vérifier les histogrammes créés
SELECT SCHEMA_NAME, TABLE_NAME, COLUMN_NAME,
       JSON_EXTRACT(HISTOGRAM, '$."histogram-type"') AS type
FROM information_schema.COLUMN_STATISTICS WHERE SCHEMA_NAME = 'lab_mysql';
```

> [!info] MySQL 8.4 : mise à jour automatique des histogrammes
> `UPDATE HISTOGRAM ... AUTO UPDATE` marque un histogramme pour un rafraîchissement automatique dès que les statistiques persistantes InnoDB sont recalculées, sans nécessiter un nouveau `ANALYZE TABLE` manuel :
> ```sql
> ANALYZE TABLE lab_mysql.logs UPDATE HISTOGRAM ON action AUTO UPDATE;
> ```

## OPTIMIZE TABLE : récupérer l'espace

Après de nombreux `DELETE`/`UPDATE`, les tables InnoDB conservent l'espace libéré dans leurs pages sans le rendre au système de fichiers.

```sql
OPTIMIZE TABLE lab_mysql.logs;
-- "Table does not support optimize, doing recreate + analyze instead" — normal pour InnoDB
```

> [!warning] Une reconstruction complète, pas une opération légère
> Pour InnoDB, `OPTIMIZE TABLE` exécute en coulisse un `ALTER TABLE ... FORCE` + `ANALYZE TABLE`, équivalent à une reconstruction complète : une **copie temporaire** de la table est créée. Prévoir de l'espace disque libre équivalent à la taille de la table ; l'opération pose un verrou metadata mais autorise les lectures/écritures DML pendant la reconstruction (Online DDL) ; sur une table de plusieurs Go, l'opération peut durer longtemps.

## CHECK TABLE et mysqlcheck : vérifier l'intégrité

```sql
CHECK TABLE lab_mysql.clients, lab_mysql.commandes, lab_mysql.logs;
```

> [!warning] `CHECK TABLE` peut bloquer d'autres threads, voire arrêter le serveur
> Sur une table InnoDB volumineuse, `CHECK TABLE` peut bloquer d'autres threads ; en cas de corruption détectée, il peut aussi provoquer l'arrêt du serveur pour éviter la propagation de l'erreur. Pour vérifier un fichier `.ibd` hors ligne sans ce risque, utiliser `innochecksum` à la place.

```bash
mysqlcheck -u root -p lab_mysql                        # vérifier toutes les tables d'une base
mysqlcheck -u root -p --optimize lab_mysql              # optimiser en masse
mysqlcheck -u root -p --all-databases --analyze         # analyser toutes les bases
```

## Cas particuliers

> [!tip] Trois outils, trois moments différents
> `ANALYZE TABLE` (statistiques) s'exécute couramment après un chargement massif ; `OPTIMIZE TABLE` (espace) se réserve aux heures creuses vu son coût ; `CHECK TABLE` (intégrité) sert de premier diagnostic face à une corruption suspectée, pas en routine.

## Pour aller plus loin

Pour une supervision continue plutôt que des vérifications ponctuelles, voir [[MySQL 20 — Monitoring Prometheus-Grafana & dépannage]].

Sources : [Superviser et maintenir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/monitoring-maintenance/)
