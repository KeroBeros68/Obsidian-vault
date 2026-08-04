#bdd #sqlite #pièges #erreurs #debugging

## 🪤 Piège 1 — Copier le fichier `.db` pendant une écriture active

```bash
# ❌ Le fichier peut être dans un état incohérent au moment de la copie
cp inventaire.db inventaire-backup.db
```

```bash
# ✅ Copie garantie cohérente, même avec des lectures en cours
sqlite3 inventaire.db ".backup inventaire-backup.db"
```

> [!warning] Un `cp` brut n'est sûr que si aucun processus n'écrit au moment de la copie
> En cas de doute, toujours `.backup`, `VACUUM INTO` ou `sqlite3_rsync`. Voir [[SQLite 05 — Sauvegarde et restauration]].

---

## 🪤 Piège 2 — Supprimer manuellement `*.db-wal` / `*.db-shm`

```bash
# ❌ Ces fichiers contiennent des modifications pas encore intégrées au .db principal
rm inventaire.db-wal inventaire.db-shm
```

> [!warning] Perte de données en mode WAL
> En mode WAL, les écritures récentes résident dans `.db-wal` tant qu'un checkpoint ne les a pas réintégrées dans le fichier `.db`. Les supprimer revient à perdre ces écritures. Toujours copier les trois fichiers ensemble. Voir [[SQLite 03 — Verrous, journalisation & mode WAL]].

---

## 🪤 Piège 3 — Stocker la base sur un montage réseau (NFS/CIFS/SMB)

```bash
# ❌ Les verrous POSIX ne sont pas fiables sur ces systèmes de fichiers
/mnt/nfs-share/inventaire.db
```

> [!warning] Risque réel de corruption
> SQLite peut corrompre la base sur un montage NFS car son mécanisme de verrouillage de fichier dépend de garanties que NFS n'offre pas de façon fiable. Voir [[SQLite 07 — Bonnes pratiques admin]].

---

## 🪤 Piège 4 — Croire que le mode WAL autorise plusieurs écrivains

```sql
-- ❌ WAL activé ne change rien à cette contrainte
PRAGMA journal_mode=WAL;
-- deux processus tentent d'écrire simultanément → l'un reçoit SQLITE_BUSY
```

> [!warning] WAL améliore la lecture concurrente, pas l'écriture concurrente
> Un seul processus peut écrire à la fois, avec ou sans WAL. Une application nécessitant plusieurs écrivains simultanés doit migrer vers un SGBD serveur — voir [[SQLite 06 — Limites en production & SQLite vs PostgreSQL]].

---

## 🪤 Piège 5 — Construire une requête SQL par concaténation en Python

```python
# ❌ Injection SQL possible
conn.execute(f"SELECT * FROM serveurs WHERE hostname = '{user_input}'")
```

```python
# ✅ Paramètre lié
conn.execute("SELECT * FROM serveurs WHERE hostname = ?", (user_input,))
```

> [!warning] Vrai même pour un script d'administration interne
> Une variable insérée par f-string ou `%` dans du SQL reste une injection SQL, y compris dans un outil CLI non exposé publiquement. Voir [[SQLite 02 — SQLite avec Python]].

---

## 🪤 Piège 6 — Fichier de base en `chmod 666`/`777`

```bash
# ❌ N'importe quel utilisateur du système peut lire toutes les données
chmod 666 inventaire.db
```

> [!warning] SQLite n'a aucune authentification
> La sécurité repose entièrement sur les permissions du fichier — un `chmod 600` avec le bon propriétaire applicatif est la seule protection. Voir [[SQLite 07 — Bonnes pratiques admin]].

---

## 🪤 Piège 7 — `VACUUM` dans une boucle applicative

```sql
-- ❌ Verrouille la base et consomme de l'espace disque supplémentaire à chaque appel
VACUUM;  -- exécuté après chaque transaction
```

> [!warning] `VACUUM` est une opération de maintenance ponctuelle
> À réserver à une tâche planifiée (cron), jamais à un chemin d'exécution fréquent. `PRAGMA optimize` (mise à jour des statistiques) est en revanche léger et adapté à une exécution avant fermeture de connexion. Voir [[SQLite 07 — Bonnes pratiques admin]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Copie brute du `.db` pendant une écriture | `.backup`, `VACUUM INTO` ou `sqlite3_rsync` |
| Suppression manuelle de `.db-wal`/`.db-shm` | Ne jamais les toucher, copier les trois fichiers ensemble |
| Base stockée sur montage NFS/CIFS/SMB | Stockage local uniquement |
| Attente de plusieurs écrivains simultanés grâce à WAL | WAL n'améliore que la lecture concurrente — migrer vers un SGBD serveur si besoin |
| Requête SQL construite par concaténation | Toujours des paramètres liés (`?`) |
| Fichier de base en `chmod 666`/`777` | `chmod 600` + propriétaire applicatif |
| `VACUUM` exécuté en boucle applicative | Réserver à une tâche planifiée, utiliser `PRAGMA optimize` au besoin |
