#bdd #sqlite #fondamentaux #cli

## Créer une base et une table

`sqlite3 inventaire.db` ouvre une session interactive sur le fichier indiqué, qu'il existe déjà ou non — le `CREATE TABLE` saisi ensuite à l'invite `sqlite>` est ce qui matérialise réellement la base.

```bash
sqlite3 inventaire.db
```

```sql
CREATE TABLE serveurs (
    id INTEGER PRIMARY KEY,
    hostname TEXT NOT NULL UNIQUE,
    ip TEXT NOT NULL,
    os TEXT NOT NULL,
    cpu_cores INTEGER NOT NULL,
    ram_gb REAL NOT NULL,
    env TEXT NOT NULL DEFAULT 'dev'
);
```

> [!warning] Le fichier n'apparaît qu'à la première écriture réelle
> `sqlite3 inventaire.db` ne crée pas immédiatement le fichier sur le disque. Une session ouverte puis quittée sans rien exécuter ne laisse aucune trace.

## Insérer des données

Un seul `INSERT` peut porter plusieurs lignes séparées par des virgules — nettement plus rapide qu'une commande par enregistrement, car SQLite regroupe le tout dans une seule transaction :

```sql
INSERT INTO serveurs (hostname, ip, os, cpu_cores, ram_gb, env) VALUES
    ('web-prod-01', '10.0.1.10', 'Debian 12', 4, 8.0, 'prod'),
    ('db-prod-01',  '10.0.1.20', 'RHEL 9',    8, 32.0, 'prod'),
    ('ci-runner-01','10.0.2.10', 'Ubuntu 24.04', 2, 4.0, 'ci'),
    ('dev-box-01',  '10.0.3.10', 'Fedora 40', 4, 16.0, 'dev');
```

La contrainte `UNIQUE` sur `hostname` rejette un doublon — ce qui rend le script rejouable sans créer d'entrées en double.

## Requêter

SQLite accepte la syntaxe SQL standard :

```sql
-- Serveurs de production triés par RAM
SELECT hostname, os, ram_gb FROM serveurs
WHERE env = 'prod' ORDER BY ram_gb DESC;
```

```
db-prod-01|RHEL 9|32.0
web-prod-01|Debian 12|8.0
```

> [!info] Sortie par défaut : compacte, pas lisible
> Les colonnes sont séparées par `|`, sans en-têtes — commode à découper dans un script, peu lisible à l'écran. Les commandes dot ci-dessous permettent de reformater.

## Commandes dot utiles

| Commande | Rôle |
|----------|------|
| `.tables` | Lister les tables |
| `.schema serveurs` | Afficher le DDL d'une table |
| `.headers on` | Afficher les noms de colonnes |
| `.mode column` | Sortie en colonnes alignées |
| `.mode csv` | Sortie CSV |
| `.mode json` | Sortie JSON |
| `.dump` | Exporter toute la base en SQL |
| `.quit` | Quitter |

```sql
.headers on
.mode column
SELECT hostname, os, ram_gb, env FROM serveurs;
```

```
hostname      os            ram_gb  env
------------  ------------  ------  ----
web-prod-01   Debian 12     8.0     prod
db-prod-01    RHEL 9        32.0    prod
ci-runner-01  Ubuntu 24.04  4.0     ci
dev-box-01    Fedora 40     16.0    dev
```

## Pour aller plus loin

La manipulation de la même base depuis un script est couverte dans [[SQLite 02 — SQLite avec Python]] ; le fonctionnement interne du fichier (verrous, journalisation) dans [[SQLite 03 — Verrous, journalisation & mode WAL]].

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
