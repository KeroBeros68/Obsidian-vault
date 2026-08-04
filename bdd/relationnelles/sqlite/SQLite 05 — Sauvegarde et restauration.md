#bdd #sqlite #intermédiaire #sauvegarde

Sauvegarder une base SQLite paraît trivial puisqu'il s'agit d'un fichier — et c'est précisément le piège. Une copie brute réalisée pendant qu'un processus écrit produit un fichier incohérent, inutilisable au moment où le besoin s'en fait sentir. Les méthodes ci-dessous garantissent au contraire une copie cohérente.

## Sauvegarde en CLI

La commande `.backup` effectue une copie cohérente de la base, même si des lectures sont en cours :

```bash
sqlite3 inventaire.db ".backup inventaire-backup.db"
```

Ou exporter un dump SQL complet, lisible, versionnable avec Git, portable entre architectures :

```bash
sqlite3 inventaire.db ".dump" > inventaire-dump.sql
```

## `VACUUM INTO`

Crée une copie compactée de la base dans un nouveau fichier, en éliminant la fragmentation :

```sql
VACUUM INTO '/backup/inventaire-compact.db';
```

## `sqlite3_rsync` : sauvegarde distante incrémentale

Disponible depuis la version 3.50, synchronise une base vers un serveur distant en ne transmettant que les pages modifiées — beaucoup plus efficace qu'un `scp` du fichier entier sur une base volumineuse :

```bash
sqlite3_rsync inventaire.db user@backup-srv:/backup/inventaire.db
```

## Restauration

La première action, quelle que soit la méthode, reste la même : arrêter tout ce qui accède à la base, sous peine d'écraser un fichier en cours d'utilisation.

```bash
# Depuis un fichier .db copié avec .backup ou VACUUM INTO
cp inventaire-backup.db inventaire.db

# Depuis un dump SQL
sqlite3 inventaire-restored.db < inventaire-dump.sql
```

Précautions avant restauration :

- Stopper tous les processus qui accèdent à la base.
- Si WAL est activé, vérifier l'absence de fichiers `*.db-wal` orphelins dans le répertoire cible — les supprimer si le `.db` restauré est un backup propre.
- Ne jamais copier un fichier `.db` brut pendant qu'un processus écrit dedans — utiliser `.backup`, `VACUUM INTO` ou `sqlite3_rsync`.
- Vérifier l'intégrité après restauration :

```bash
sqlite3 inventaire.db "PRAGMA integrity_check;"
```

```
ok
```

> [!warning] `cp` brut n'est sûr que si aucun processus n'écrit au moment de la copie
> En cas de doute, toujours utiliser `.backup` ou `VACUUM INTO` plutôt qu'un `cp` direct — voir [[SQLite 03 — Verrous, journalisation & mode WAL]] pour comprendre pourquoi une copie en cours d'écriture est incohérente.

## Pour aller plus loin

`PRAGMA integrity_check` revient aussi comme vérification de routine (hors contexte de restauration) dans [[SQLite 07 — Bonnes pratiques admin]].

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
