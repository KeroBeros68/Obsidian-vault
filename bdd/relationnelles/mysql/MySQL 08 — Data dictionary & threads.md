#bdd #mysql #architecture #avancé

## Le data dictionary : des métadonnées transactionnelles depuis MySQL 8.0

Avant MySQL 8.0, les métadonnées des tables (colonnes, index, types) étaient stockées dans des fichiers `.frm`, un par table, dans un format propriétaire non transactionnel — un crash pendant un DDL (`ALTER TABLE`, `DROP TABLE`) pouvait créer des incohérences entre ces fichiers et les données InnoDB.

Depuis MySQL 8.0, le **data dictionary** est stocké dans des tables InnoDB transactionnelles, regroupées dans un unique fichier `mysql.ibd` :

```bash
ls /var/lib/mysql/mysql.ibd
```

Ce fichier contient l'intégralité des métadonnées (tables, colonnes, index, contraintes, charset, collation, routines, vues) pour toutes les bases de l'instance. Conséquence directe : les opérations DDL sont désormais **atomiques** — un `DROP TABLE` portant sur plusieurs tables réussit ou échoue entièrement, sans état intermédiaire corrompu possible.

> [!warning] La migration depuis les fichiers `.frm` est irréversible
> Une migration depuis MySQL 5.7 déclenche `mysql_upgrade` (ou `--upgrade=AUTO` depuis MySQL 8.0.16), qui convertit automatiquement les métadonnées `.frm` vers le data dictionary InnoDB. Toujours garder une sauvegarde complète avant cette migration.

## Les threads de connexion

Chaque client connecté obtient un **thread de connexion** (foreground thread) dédié, qui exécute ses requêtes et gère ses transactions. À la déconnexion, ce thread peut être recyclé via le **thread cache** plutôt que détruit :

```sql
SHOW VARIABLES LIKE 'thread_cache_size';
-- 9 threads gardés en réserve par défaut
```

Garder des threads en réserve réduit le coût de création pour des applications qui ouvrent et ferment beaucoup de connexions courtes.

```sql
SHOW PROCESSLIST;
-- une ligne par thread de connexion actif
```

## Les threads d'arrière-plan d'InnoDB

En plus des threads de connexion, InnoDB fait tourner des threads de maintenance continue :

| Thread | Rôle |
|--------|------|
| Master thread | Coordonne les opérations d'arrière-plan, déclenche les flushes |
| Page cleaner | Écrit les dirty pages du buffer pool vers les fichiers `.ibd` |
| Purge thread | Supprime les anciennes versions de lignes (MVCC) devenues inutiles |
| Log writer / Log flusher | Écrivent puis synchronisent le redo log buffer sur disque |
| Checkpoint | Marque jusqu'où les données du redo log sont écrites sur disque |

> [!info] Ces threads sont invisibles avec `ps`
> Contrairement à PostgreSQL, où les workers sont des processus visibles individuellement avec `ps aux`, les threads InnoDB sont internes au seul processus `mysqld` — pour les inspecter, interroger `performance_schema.threads` plutôt que chercher des PID supplémentaires.

```sql
SELECT name, type, processlist_command FROM performance_schema.threads
WHERE type = 'BACKGROUND' ORDER BY name LIMIT 15;
```

## Pour aller plus loin

Le choix entre les tracks de version LTS et Innovation, ainsi que l'arborescence complète des fichiers d'une installation, sont couverts dans [[MySQL 09 — Versions (LTS vs Innovation) & arborescence des fichiers]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
