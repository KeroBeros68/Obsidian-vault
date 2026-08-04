#bdd #sqlite #intermédiaire #strict #json

SQLite n'est plus le « petit moteur SQL des années 2000 ». Les versions récentes intègrent des fonctionnalités qui changent la façon de l'utiliser — attention au seuil de version pour chacune, voir [[SQLite 00 — Installation]].

## STRICT tables : du vrai typage

Par défaut, SQLite est très permissif : insérer du texte dans une colonne `INTEGER` ne produit pas d'erreur. Les STRICT tables (depuis la version 3.37) changent ce comportement :

```sql
CREATE TABLE serveurs (
    id INTEGER PRIMARY KEY,
    hostname TEXT NOT NULL,
    cpu_cores INTEGER NOT NULL,
    ram_gb REAL NOT NULL
) STRICT;
```

Avec `STRICT`, `INSERT INTO serveurs (hostname, cpu_cores, ram_gb) VALUES ('srv01', 'quatre', 8.0)` échoue avec une erreur de type — le comportement attendu pour des données d'infrastructure.

## Colonnes générées

Depuis la version 3.31, une colonne générée calcule sa valeur automatiquement à partir d'autres colonnes :

```sql
CREATE TABLE disques (
    id INTEGER PRIMARY KEY,
    device TEXT NOT NULL,
    total_mb INTEGER NOT NULL,
    used_mb INTEGER NOT NULL,
    pct_used REAL GENERATED ALWAYS AS (100.0 * used_mb / total_mb) STORED
) STRICT;

INSERT INTO disques (device, total_mb, used_mb) VALUES ('/dev/sda1', 102400, 87040);
SELECT device, pct_used FROM disques;
```

```
/dev/sda1|85.0
```

`pct_used` est calculée automatiquement — pas besoin de la maintenir manuellement à chaque `UPDATE`.

## JSON et JSONB natifs

Le JSON est géré nativement depuis la version 3.38, JSONB depuis la 3.45 — utile quand une partie des données est semi-structurée :

```sql
CREATE TABLE events (
    id INTEGER PRIMARY KEY,
    timestamp TEXT NOT NULL DEFAULT (datetime('now')),
    source TEXT NOT NULL,
    payload TEXT NOT NULL  -- stocke du JSON
) STRICT;

INSERT INTO events (source, payload) VALUES
    ('prometheus', '{"alert":"DiskFull","host":"web-prod-01","pct":95}'),
    ('prometheus', '{"alert":"HighCPU","host":"db-prod-01","pct":88}');

-- Extraire le champ "host" du JSON
SELECT source,
       json_extract(payload, '$.host') AS host,
       json_extract(payload, '$.alert') AS alert
FROM events;
```

```
prometheus|web-prod-01|DiskFull
prometheus|db-prod-01|HighCPU
```

## Pour aller plus loin

Ce même comparatif JSON revient dans [[SQLite 06 — Limites en production & SQLite vs PostgreSQL]] face aux capacités JSONB/GIN de PostgreSQL.

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
