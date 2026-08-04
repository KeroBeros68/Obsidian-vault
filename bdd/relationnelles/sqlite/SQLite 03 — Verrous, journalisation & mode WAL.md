#bdd #sqlite #intermédiaire #wal #concurrence

## Le fichier, le verrou, le journal

SQLite stocke tout dans un fichier unique. Quand un processus écrit, SQLite verrouille **le fichier entier** — pas une ligne, pas une table — empêchant les autres processus d'écrire en même temps.

Par défaut, SQLite utilise un **journal rollback** : avant chaque écriture, les pages modifiées sont copiées dans un fichier `.db-journal`. En cas de crash, ce journal restaure l'état précédent. Le problème : pendant une écriture, les lectures sont elles aussi bloquées.

C'est ce comportement que le mode WAL change radicalement.

## Le mode WAL (Write-Ahead Logging)

Le WAL inverse le mécanisme : au lieu de copier les anciennes pages avant d'écrire, SQLite écrit les modifications dans un fichier séparé (`*.db-wal`). Les lecteurs continuent à lire l'ancienne version du fichier pendant que l'écrivain travaille.

### Activer WAL

```sql
PRAGMA journal_mode=WAL;
```

```python
with sqlite3.connect("inventaire.db") as conn:
    conn.execute("PRAGMA journal_mode=WAL")
```

> [!warning] Vérifier la valeur retournée
> Le `PRAGMA` retourne le mode effectivement appliqué — `wal` en cas de succès. Le passage en WAL échoue silencieusement sur certains systèmes de fichiers réseau (voir [[SQLite 07 — Bonnes pratiques admin]] sur NFS).

Le mode est persistant : il est écrit dans le fichier de la base, pas besoin de le réactiver à chaque connexion.

### Ce que WAL change concrètement

| Aspect | Journal rollback (défaut) | WAL |
|--------|-----------------------------|-----|
| Lectures pendant une écriture | Bloquées | Autorisées |
| Plusieurs lecteurs simultanés | Oui | Oui |
| Plusieurs écrivains simultanés | Non | Non |
| Fichiers supplémentaires | `.db-journal` (temporaire) | `.db-wal` + `.db-shm` (persistants) |
| Performance en lecture | Bonne | Meilleure |

La seule différence décisive porte sur les lectures pendant une écriture — c'est là que WAL apporte un gain réel. Sur la concurrence en écriture, les deux modes sont identiques : WAL ne lève pas la limitation fondamentale d'un seul écrivain à la fois. Il supprime seulement le blocage des lecteurs, ce qui améliore sensiblement les cas d'usage typiques (beaucoup de lectures, peu d'écritures).

> [!warning] Ne jamais supprimer `*.db-wal` / `*.db-shm` manuellement
> Ces fichiers font partie intégrante de la base en mode WAL. Pour copier ou déplacer la base, copier les trois fichiers ensemble.

## Pour aller plus loin

Cette limitation à un seul écrivain, même en WAL, est l'un des critères qui déclenchent une migration vers PostgreSQL — voir [[SQLite 06 — Limites en production & SQLite vs PostgreSQL]]. Pour sauvegarder une base avec WAL activé sans la corrompre, voir [[SQLite 05 — Sauvegarde et restauration]].

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
