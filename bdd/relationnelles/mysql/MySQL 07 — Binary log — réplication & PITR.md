#bdd #mysql #réplication #avancé

## Deux journaux, deux rôles distincts

MySQL utilise deux systèmes de journalisation qui ne se substituent pas l'un à l'autre :

| | Redo log | Binary log (binlog) |
|---|----------|------------------------|
| Géré par | InnoDB (moteur de stockage) | MySQL Server (couche SQL) |
| Contenu | Modifications physiques des pages InnoDB | Événements logiques (`INSERT`, `UPDATE`, `DELETE`, DDL) |
| Usage | Crash recovery | Réplication + PITR (restauration à un instant précis) |
| Fichiers | `#innodb_redo/#ib_redo*` | `binlog.000001`, `binlog.000002`... |
| Rétention | Court terme (entre checkpoints) | Configurable (`binlog_expire_logs_seconds`, défaut 30 jours) |

Le redo log (voir [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]]) est **interne** à InnoDB, pour qu'il puisse se relever après un crash. Le binary log est **global**, utilisé pour envoyer les changements aux réplicas et pour restaurer les données à un instant précis.

## Les deux journaux travaillent ensemble

Lors d'un `INSERT`, MySQL écrit d'abord dans le redo log (crash recovery), puis dans le binary log (réplication). Un mécanisme **XA (two-phase commit)** garantit leur cohérence : si le binlog ne confirme pas l'écriture, la transaction est annulée dans le redo log.

## Les trois formats du binary log

| Format | Enregistre | Avantage | Inconvénient |
|--------|------------|----------|-------------------|
| `ROW` | L'image avant/après de chaque ligne modifiée | Déterministe, pas d'ambiguïté | Binlog plus volumineux pour les `UPDATE` massifs |
| `STATEMENT` | La requête SQL elle-même | Binlog compact | Non déterministe (`NOW()`, `UUID()`, fonctions aléatoires) |
| `MIXED` | STATEMENT quand possible, ROW quand nécessaire | Compromis | Comportement parfois imprévisible |

```sql
SHOW VARIABLES LIKE 'binlog_format';
-- ROW (par défaut depuis MySQL 5.7.7, format recommandé)
```

`ROW` garantit une réplication déterministe : le réplica applique exactement les mêmes modifications que la source, quelles que soient les fonctions utilisées dans la requête d'origine.

```bash
mysqlbinlog --verbose /var/lib/mysql/binlog.000002 | head -40
```

Le binary log contient des événements structurés : `Query_event` (DDL), `Write_rows_event` (INSERT), `Update_rows_event` (UPDATE), `Delete_rows_event` (DELETE).

## Cas particuliers

> [!warning] STATEMENT peut désynchroniser un réplica silencieusement
> Une requête utilisant `NOW()` ou une fonction aléatoire, enregistrée en `STATEMENT`, est rejouée sur le réplica à un instant différent — produisant un résultat différent de la source, sans erreur visible. `ROW` élimine ce risque en enregistrant le résultat final plutôt que la requête.

> [!info] MySQL 8.4 LTS active le binlog par défaut
> Contrairement aux versions antérieures qui nécessitaient `--log-bin` explicite, MySQL 8.4 LTS active le binary log par défaut — voir [[MySQL 09 — Versions (LTS vs Innovation) & arborescence des fichiers]] pour les autres changements notables de cette version.

## Pour aller plus loin

Les métadonnées de tables (data dictionary) et les threads MySQL sont couverts dans [[MySQL 08 — Data dictionary & threads]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
