#bdd #sqlite #fondamentaux #python

## Le module `sqlite3` de la bibliothèque standard

Inclus dans Python, aucun `pip install` nécessaire — c'est le moyen le plus courant de manipuler SQLite depuis un script d'administration.

## Connexion et requête de base

```python
import sqlite3
from pathlib import Path

db_path = Path("inventaire.db")

with sqlite3.connect(db_path) as conn:
    conn.row_factory = sqlite3.Row  # accès par nom de colonne
    cursor = conn.execute(
        "SELECT hostname, os, ram_gb FROM serveurs WHERE env = ?",
        ("prod",),
    )
    for row in cursor:
        print(f"{row['hostname']:20s} {row['os']:15s} {row['ram_gb']:.0f} Go")
```

```
web-prod-01          Debian 12       8 Go
db-prod-01           RHEL 9          32 Go
```

Trois détails rendent ce script robuste :

- **`with sqlite3.connect(...)`** — le context manager garantit que la transaction est commitée (ou rollbackée en cas d'erreur) et que la connexion est fermée proprement.
- **`?` (paramètres liés)** — ne jamais injecter une variable dans une chaîne SQL avec des f-strings ou `%s`.
- **`row_factory = sqlite3.Row`** — accès aux colonnes par nom (`row['hostname']`) plutôt que par index numérique.

## Créer une table et insérer depuis Python

```python
import sqlite3

with sqlite3.connect("alertes.db") as conn:
    conn.execute("""
        CREATE TABLE IF NOT EXISTS alertes (
            id INTEGER PRIMARY KEY,
            timestamp TEXT NOT NULL DEFAULT (datetime('now')),
            host TEXT NOT NULL,
            severity TEXT NOT NULL CHECK (severity IN ('info','warn','crit')),
            message TEXT NOT NULL
        )
    """)
    conn.executemany(
        "INSERT INTO alertes (host, severity, message) VALUES (?, ?, ?)",
        [
            ("web-prod-01", "crit", "Disque / à 95%"),
            ("db-prod-01",  "warn", "Réplication en retard de 120s"),
            ("ci-runner-01","info", "Build #1847 terminé"),
        ],
    )
```

- `CREATE TABLE IF NOT EXISTS` rend le script idempotent — relançable sans erreur sur une base déjà initialisée.
- `executemany` insère toute une liste en une seule transaction, ce qui divise nettement le temps d'exécution sur des volumes importants.
- La contrainte `CHECK` sur `severity` refuse toute valeur hors des trois autorisées, directement au niveau de la base.

> [!warning] Ne jamais construire une requête SQL par concaténation de chaînes
> ```python
> # ❌ DANGEREUX : injection SQL possible
> conn.execute(f"SELECT * FROM serveurs WHERE hostname = '{user_input}'")
>
> # ✅ SÛR : paramètre lié
> conn.execute("SELECT * FROM serveurs WHERE hostname = ?", (user_input,))
> ```

## Pour aller plus loin

Le fonctionnement du fichier sous-jacent (verrous, mode WAL pour la concurrence de lecture) est détaillé dans [[SQLite 03 — Verrous, journalisation & mode WAL]].

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
