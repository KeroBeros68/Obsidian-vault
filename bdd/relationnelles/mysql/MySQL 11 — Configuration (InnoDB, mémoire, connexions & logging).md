#bdd #mysql #intermédiaire #configuration

## Où se trouve la configuration

### Hiérarchie des fichiers

```bash
mysqld --verbose --help 2>/dev/null | grep -A1 "Default options"
```

```
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf
```

Sur Debian/Ubuntu, l'organisation typique après installation depuis le dépôt Oracle :

```
/etc/mysql/
├── my.cnf                          → point d'entrée, inclut conf.d/ et mysql.conf.d/
├── conf.d/mysql.cnf                 → options du client mysql
└── mysql.conf.d/mysqld.cnf          → configuration du serveur mysqld

/var/lib/mysql/mysqld-auto.cnf       → écrit par SET PERSIST, prioritaire sur les fichiers manuels
```

Sur RHEL/Rocky, tout est dans un fichier unique `/etc/my.cnf` — voir [[MySQL 00 — Installation]] pour les chemins par distribution.

### Sections `[mysqld]`, `[client]`, `[mysql]`

| Section | S'applique à | Exemples de paramètres |
|---------|--------------|-------------------------|
| `[mysqld]` | Le serveur | `innodb_buffer_pool_size`, `max_connections`, `bind_address` |
| `[client]` | Tous les clients (`mysql`, `mysqldump`, `mysqlimport`) | `port`, `socket`, `default-character-set` |
| `[mysql]` | Le client `mysql` uniquement | `prompt`, `pager`, `auto-rehash` |

`[mysqld]` est la section à modifier pour configurer le serveur lui-même.

### `mysqld-auto.cnf` : où `SET PERSIST` écrit

`SET PERSIST innodb_buffer_pool_size = 1073741824;` écrit la valeur dans `/var/lib/mysql/mysqld-auto.cnf`, un fichier **JSON** (pas le format INI classique) :

```bash
cat /var/lib/mysql/mysqld-auto.cnf | python3 -m json.tool | head -20
```

```json
{
    "Version": 2,
    "mysql_dynamic_parse_early_variables": {
        "innodb_buffer_pool_size": {
            "Value": "1073741824",
            "Metadata": { "Host": "", "User": "root", "Timestamp": 1744530000 }
        }
    }
}
```

`mysqld-auto.cnf` est lu en dernier : ses valeurs sont prioritaires sur `mysqld.cnf` et `my.cnf`.

> [!warning] Ne jamais modifier `mysqld-auto.cnf` à la main
> Ce fichier est géré par MySQL. Une syntaxe JSON cassée empêchera le serveur de démarrer. Utiliser `SET PERSIST` pour ajouter des valeurs, `RESET PERSIST variable_name` pour les supprimer.

## Appliquer les changements

| Méthode | Effet immédiat | Persiste au redémarrage | Usage |
|---------|-----------------|--------------------------|-------|
| `SET GLOBAL` | Oui | Non | Test rapide, changement temporaire |
| `SET PERSIST` | Oui | Oui (écrit dans `mysqld-auto.cnf`) | Changement définitif sur variable dynamique |
| `SET PERSIST_ONLY` | Non | Oui | Variable statique — appliqué au prochain restart |
| `RESET PERSIST [variable]` | — | Supprime la valeur persistée | Retour au défaut ou au fichier `.cnf` |

```sql
SET GLOBAL slow_query_log = ON;              -- non persistant
SET PERSIST slow_query_log = ON;             -- persistant
SET PERSIST_ONLY innodb_buffer_pool_instances = 4;  -- statique, effectif au restart
RESET PERSIST slow_query_log;                -- supprime cette valeur persistée
RESET PERSIST;                                -- supprime TOUTES les variables persistées
```

### Variables dynamiques vs statiques

| Type | Modifiable à chaud ? | Méthode |
|------|------------------------|---------|
| Dynamique | Oui | `SET GLOBAL` ou `SET PERSIST` |
| Statique | Non | Modifier `mysqld.cnf` ou `SET PERSIST_ONLY`, puis redémarrer |

En pratique, un `SET GLOBAL` qui échoue avec `ERROR 1238 (HY000): Variable 'xxx' is a read only variable` signale une variable statique.

> [!tip] `innodb_buffer_pool_size` est dynamique depuis MySQL 5.7
> Contrairement à PostgreSQL où `shared_buffers` exige un restart, le buffer pool InnoDB se redimensionne à chaud avec `SET GLOBAL`/`SET PERSIST` — MySQL le réalloue progressivement. Surveiller `Innodb_buffer_pool_resize_status` pendant l'opération.

> [!info] `performance_schema.variables_info`
> Indique la source de la valeur courante (`COMPILED`, `GLOBAL`, `PERSISTED`, `DYNAMIC`), le moment et l'utilisateur du dernier changement — mais pas si la variable est dynamique ou statique.

## Configurer la mémoire InnoDB

### `innodb_buffer_pool_size` : la règle des 50-70 %

Cache mémoire central d'InnoDB pour les pages de données et d'index — le paramètre le plus impactant. Valeur par défaut : `128 Mo`, très insuffisant en production.

```sql
SET PERSIST innodb_buffer_pool_size = 1073741824;  -- 1 Go, exprimé en octets avec SET
```

```ini
[mysqld]
innodb_buffer_pool_size = 1G   -- suffixes M/G acceptés en fichier .cnf, pas avec SET
```

Dimensionner entre 50 et 70 % de la RAM totale sur un serveur dédié — le reste sert au thread cache, aux buffers de tri, aux tables temporaires, au cache du binary log et au page cache de l'OS.

```sql
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool%';
```

| Compteur | Signification |
|----------|---------------|
| `Innodb_buffer_pool_read_requests` | Lectures servies depuis le buffer pool (cache hit) |
| `Innodb_buffer_pool_reads` | Lectures nécessitant un accès disque (cache miss) |
| `Innodb_buffer_pool_pages_total` | Pages totales dans le buffer pool |
| `Innodb_buffer_pool_pages_free` | Pages libres (0 = buffer pool plein) |

```sql
SELECT ROUND((1 - (Reads / Read_requests)) * 100, 2) AS buffer_pool_hit_ratio
FROM (
  SELECT
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads') AS Reads,
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests') AS Read_requests
) AS stats;
```

Objectif : hit ratio > 99 %. En dessous de 95 %, augmenter le buffer pool.

### `innodb_buffer_pool_instances`, `innodb_log_buffer_size`, `innodb_flush_method`

MySQL 8.4 ajuste automatiquement `innodb_buffer_pool_instances` selon la taille totale (1 instance par Go, plafonné à 64) — rarement à configurer manuellement.

`innodb_log_buffer_size` (buffer avant écriture des entrées de redo log sur disque) : `64 Mo` par défaut en 8.4 (contre 16 Mo en 8.0). À augmenter pour de grosses transactions (`LOAD DATA`, insertions massives) :

```sql
SET PERSIST innodb_log_buffer_size = 134217728;  -- 128 Mo
```

`innodb_flush_method` contrôle l'écriture disque des données et logs :

| Valeur | Comportement | Recommandation |
|--------|--------------|-----------------|
| `O_DIRECT` | Bypass du cache OS (défaut 8.4) | Recommandé, évite la double bufferisation |
| `fsync` | Passe par le cache OS puis fsync | Ancien défaut, moins efficace |
| `O_DIRECT_NO_FSYNC` | Comme `O_DIRECT` sans fsync pour les redo logs | Risque de perte sans cache protégé par batterie |

## Les nouveaux défauts InnoDB de MySQL 8.4

| Paramètre | Ancien défaut (8.0) | Nouveau défaut (8.4) | Impact |
|-----------|----------------------|------------------------|--------|
| `innodb_io_capacity` | 200 | 10 000 | Écriture plus agressive des pages sales |
| `innodb_io_capacity_max` | 2× io_capacity | 2× io_capacity (= 20 000) | Plafond d'I/O plus élevé |
| `innodb_change_buffering` | `all` | `none` | Change buffer désactivé |
| `innodb_adaptive_hash_index` | `ON` | `OFF` | Hash index adaptatif désactivé |
| `innodb_flush_method` | `fsync` | `O_DIRECT` | Bypass du cache OS par défaut |
| `innodb_log_buffer_size` | 16 Mo | 64 Mo | Buffer de redo log quadruplé |
| `innodb_doublewrite_pages` | 64 | 128 | Plus de pages dans le doublewrite buffer — voir [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]] |

> [!warning] `innodb_io_capacity` suppose du SSD/NVMe
> MySQL 8.4 part du principe que le stockage moderne encaisse des milliers d'IOPS. Sur des disques mécaniques (HDD), baisser explicitement : `SET PERSIST innodb_io_capacity = 200;`.

`innodb_change_buffering=none` et `innodb_adaptive_hash_index=OFF` : ces optimisations historiques apportaient un gain devenu négligeable, voire contre-productif (contention sur sémaphores), sur les SSD modernes — les garder désactivées sauf benchmark contraire.

### `innodb_dedicated_server`

Non modifiable à chaud, à activer dans `mysqld.cnf` :

```ini
[mysqld]
innodb_dedicated_server = ON
```

Calcule automatiquement, selon la RAM détectée :

| RAM détectée | Buffer pool |
|--------------|-------------|
| < 1 Go | 128 Mio |
| 1 à 4 Go | 50 % de la RAM |
| > 4 Go | 75 % de la RAM |

Et `innodb_redo_log_capacity` selon le nombre de processeurs logiques : `(logical processors / 2) GiB`, plafonné à 16 GiB.

> [!warning] Réservé aux serveurs 100 % dédiés à MySQL
> `innodb_dedicated_server` suppose que toute la RAM appartient à MySQL. Si le serveur héberge aussi Nginx, Redis ou une application, l'auto-sizing surconsommera la mémoire — dimensionner manuellement dans ce cas.

## Configurer les connexions

### `max_connections`

Défaut `151`. Chaque connexion consomme un thread (modèle multi-thread, voir [[MySQL 01 — L'instance mysqld]]), donc une pile mémoire (`thread_stack`, 1 Mo par défaut) et ses propres buffers de session.

```sql
SET PERSIST max_connections = 300;
```

- Développement/lab : le défaut de 151 suffit
- Production : ajuster selon la charge réelle, sans dépasser 500-1000
- Au-delà de 300-500 connexions : envisager ProxySQL ou un pool de connexions applicatif

### `wait_timeout` et `interactive_timeout`

| Paramètre | S'applique à | Défaut |
|-----------|--------------|--------|
| `wait_timeout` | Connexions non interactives (application) | 28 800 s (8 h) |
| `interactive_timeout` | Connexions interactives (client `mysql`) | 28 800 s (8 h) |

8 heures d'inactivité avant fermeture est très long en production :

```sql
SET PERSIST wait_timeout = 600;           -- 10 minutes
SET PERSIST interactive_timeout = 3600;   -- 1 heure
```

### `thread_cache_size`

À la fermeture d'une connexion, MySQL peut garder le thread en cache plutôt que le détruire — la connexion suivante le réutilise. Défaut : auto-dimensionné selon `max_connections`, plafonné à 100.

```sql
SHOW GLOBAL STATUS LIKE 'Threads%';
```

Si `Threads_created` augmente rapidement par rapport à `Threads_connected`, augmenter le cache :

```sql
SET PERSIST thread_cache_size = 32;
```

## Configurer le binary log

Le binary log (binlog) enregistre toutes les modifications de données — indispensable pour la réplication et le PITR, voir [[MySQL 07 — Binary log — réplication & PITR]].

### `binlog_format`

| Format | Contenu | Recommandation |
|--------|---------|-----------------|
| `ROW` | Image avant/après de chaque ligne modifiée | Recommandé (défaut 8.4), déterministe |
| `STATEMENT` | La requête SQL originale | Risque de divergence source/réplica |
| `MIXED` | STATEMENT quand sûr, ROW sinon | Compromis rarement nécessaire |

### `binlog_expire_logs_seconds`

Défaut : `2 592 000` s (30 jours).

```sql
SET PERSIST binlog_expire_logs_seconds = 604800;  -- 7 jours
```

### `sync_binlog` et `innodb_flush_log_at_trx_commit` : durabilité

| `sync_binlog` | `innodb_flush_log_at_trx_commit` | Durabilité | Performance |
|---------------|-------------------------------------|-------------|-------------|
| 1 | 1 | Maximale (zéro perte en cas de crash) | Plus lent (2 écritures disque/commit) |
| 1 | 2 | Bonne (perte possible ≤ 1 s de binlog) | Meilleure |
| 0 | 0 | Minimale (perte possible en cas de crash) | Maximum |

```sql
SET PERSIST sync_binlog = 1;
SET PERSIST innodb_flush_log_at_trx_commit = 1;
```

Seul réglage garantissant zéro perte de données en cas de crash — recommandé en production pour des données critiques.

> [!warning] Le coût de la durabilité maximale
> `sync_binlog=1` + `innodb_flush_log_at_trx_commit=1` double les écritures disque par transaction. Impact modéré sur SSD/NVMe, potentiellement lourd sur HDD. Si le débit prime sur une perte tolérable d'1 seconde de données, passer `innodb_flush_log_at_trx_commit` à `2`.

## Configurer le logging

### Slow query log : trouver les requêtes lentes

Le paramètre le plus utile en exploitation, désactivé par défaut :

```sql
SET PERSIST slow_query_log = ON;
SET PERSIST long_query_time = 1;   -- secondes
```

| Paramètre | Défaut | Recommandation |
|-----------|--------|-----------------|
| `slow_query_log` | `OFF` | `ON`, toujours en production |
| `long_query_time` | `10` s | `1` ou `0.5` pour être réactif |
| `log_queries_not_using_indexes` | `OFF` | Mode investigation ponctuel uniquement |

> [!warning] `log_queries_not_using_indexes` avec prudence
> Peut faire grossir le slow log très vite sur une base à nombreuses petites tables. En investigation, ajouter un throttle : `SET PERSIST log_throttle_queries_not_using_indexes = 60;` (max 60 entrées/minute). Ce n'est pas un réglage de production permanent.

```bash
mysqldumpslow -s t /var/lib/mysql/mysql-lab-slow.log | head -30
```

### General log (debug uniquement)

Trace toutes les requêtes — utile en debug, dangereux en production (volume, impact performance) :

```sql
SET GLOBAL general_log = ON;   -- ne jamais persister
-- débugger le problème...
SET GLOBAL general_log = OFF;
```

### `log_output` : FILE vs TABLE

```sql
SET PERSIST log_output = 'FILE';        -- défaut, recommandé en production
SET PERSIST log_output = 'TABLE';       -- écrit dans mysql.slow_log et mysql.general_log
SET PERSIST log_output = 'FILE,TABLE';  -- les deux
```

`TABLE` facilite les requêtes SQL sur les logs mais consomme plus de ressources.

## Configurer le réseau

### `bind_address`

Défaut MySQL 8.4 : `*` (toutes interfaces, IPv4/IPv6). Sur un paquet Debian/Oracle, `bind_address = 127.0.0.1` est configuré par le paquet lui-même, pas par MySQL — voir [[MySQL 01 — L'instance mysqld]].

```ini
[mysqld]
bind_address = 0.0.0.0                       -- ouvrir à toutes les interfaces
bind_address = 127.0.0.1,192.168.122.70      -- ou plusieurs adresses précises (8.4+)
```

> [!warning] `bind_address` est statique
> Un restart est obligatoire après modification. N'ouvrir l'écoute réseau qu'après avoir configuré comptes et permissions — voir [[Manques]] (guide Sécurisation).

### `skip_name_resolve`

Par défaut, MySQL effectue une résolution DNS inversée à chaque connexion entrante — un coût inutile sur un réseau interne sans DNS fiable :

```sql
SET PERSIST_ONLY skip_name_resolve = ON;   -- nécessite un restart
```

Après activation, les comptes MySQL doivent utiliser des adresses IP plutôt que des noms d'hôtes dans les `GRANT`.

## Vérifier la configuration

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

SELECT variable_name, variable_source, set_time, set_user
FROM performance_schema.variables_info
WHERE variable_name LIKE 'innodb_buffer%';
```

`variable_source` indique l'origine : `COMPILED` (défaut de compilation), `GLOBAL` (`my.cnf`/`mysqld.cnf`), `PERSISTED` (`mysqld-auto.cnf`), `DYNAMIC` (`SET GLOBAL` de session).

```bash
sudo mysqld --validate-config
```

> [!tip] Valider avant chaque restart
> `mysqld --validate-config` vérifie la configuration sans démarrer le serveur — si un paramètre est invalide, la commande affiche l'erreur et sort en échec. À lancer systématiquement après une modification manuelle de `mysqld.cnf`, avant de redémarrer le service.

## Tableau récapitulatif : valeurs recommandées

| RAM totale | `innodb_buffer_pool_size` | `max_connections` | `innodb_log_buffer_size` | `long_query_time` |
|------------|------------------------------|----------------------|------------------------------|----------------------|
| 2 Go | 1 Go | 100-200 | 64 Mo | 1 s |
| 4 Go | 2-3 Go | 200-300 | 64 Mo | 1 s |
| 8 Go | 4-5 Go | 200-500 | 64 Mo | 0.5 s |
| 16 Go | 10-12 Go | 300-500 | 128 Mo | 0.5 s |
| 32 Go | 20-24 Go | 300-1000 | 128 Mo | 0.5 s |

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|-----------------|----------|
| Le serveur ne redémarre plus après un changement dans `mysqld.cnf` | Paramètre invalide ou syntaxe incorrecte | `mysqld --validate-config` pour identifier l'erreur ; corriger le fichier ou renommer `mysqld-auto.cnf` |
| `SET PERSIST` retourne `ERROR 3615` | `mysqld-auto.cnf` corrompu | Supprimer `/var/lib/mysql/mysqld-auto.cnf` et redémarrer |
| `SHOW VARIABLES` diffère de `mysqld.cnf` | `mysqld-auto.cnf` a priorité | `SELECT * FROM performance_schema.variables_info WHERE variable_name = 'xxx'` pour trouver la source |
| Buffer pool hit ratio < 95 % | Buffer pool trop petit | Augmenter `innodb_buffer_pool_size` |
| `Threads_created` augmente rapidement | Thread cache insuffisant | Augmenter `thread_cache_size` |
| Slow query log vide malgré `slow_query_log = ON` | `long_query_time` trop élevé (défaut 10 s) | Baisser à 1 ou 0.5 seconde |

## Pour aller plus loin

La sécurisation avancée (authentification, utilisateurs, rôles, TLS) est couverte dans [[MySQL 21 — Authentification (caching_sha2_password & authentication_policy)]] et les fiches suivantes. La réplication (GTID, source-réplica, Group Replication, InnoDB Cluster) est couverte à partir de [[MySQL 26 — Concepts de réplication & GTID]].

Sources : [Configurer MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/configuration/)
