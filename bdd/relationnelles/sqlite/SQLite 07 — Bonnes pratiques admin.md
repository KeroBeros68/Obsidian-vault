#bdd #sqlite #avancé #sécurité #maintenance

Administrer SQLite consiste surtout à compenser ce que le moteur ne fait pas : pas de contrôle d'accès, pas de supervision, pas de tâche de maintenance planifiée intégrée. Quatre points couvrent l'essentiel : protéger le fichier, détecter une corruption avant qu'elle ne devienne un incident, entretenir les performances, chiffrer les données quand elles le méritent.

## Permissions du fichier

Point le plus critique en sécurité SQLite pour un admin système : la base n'a **aucun système d'authentification**, quiconque peut lire le fichier peut lire toutes les données.

```bash
# Restreindre l'accès au propriétaire uniquement
chmod 600 inventaire.db
chown app-user:app-group inventaire.db

ls -la inventaire.db
```

```
-rw------- 1 app-user app-group 45056 avr 13 10:30 inventaire.db
```

Règles :

- Le fichier doit appartenir à l'utilisateur système qui exécute l'application.
- Jamais de `chmod 666` ou `chmod 777` sur un fichier de base de données.
- Stocker la base dans un répertoire aux permissions restreintes.

> [!warning] Éviter les montages réseau (NFS, CIFS, SMB)
> Les verrous POSIX ne sont pas fiables sur ces systèmes de fichiers — SQLite peut corrompre la base sur un montage NFS. Voir [[SQLite 03 — Verrous, journalisation & mode WAL]] pour comprendre le rôle de ces verrous.

## Vérifier l'intégrité

`PRAGMA integrity_check` parcourt l'ensemble des pages et des index, renvoie `ok` si tout est cohérent ou la liste des anomalies :

```bash
sqlite3 inventaire.db "PRAGMA integrity_check;"
```

> [!warning] Coûteux sur une base volumineuse
> L'opération peut durer plusieurs minutes et solliciter fortement le disque — à lancer en dehors des heures de charge, planifié dans un cron hebdomadaire pour détecter les corruptions tôt.

## Optimiser les performances

Deux opérations complémentaires :

```sql
-- Met à jour les statistiques dont l'optimiseur se sert pour choisir ses index
-- À exécuter avant de fermer une connexion de longue durée
PRAGMA optimize;

-- Reconstruit le fichier pour récupérer l'espace laissé par les suppressions
VACUUM;
```

> [!warning] `VACUUM` verrouille la base et demande de l'espace disque supplémentaire
> Contrairement à `PRAGMA optimize`, `VACUUM` n'a pas sa place dans une boucle applicative — c'est une opération de maintenance ponctuelle, pas un réflexe systématique.

## Chiffrement avec SQLCipher

Pour des données sensibles, SQLCipher ajoute un chiffrement AES-256 transparent :

```bash
# Installation (Debian/Ubuntu)
sudo apt install sqlcipher

# Créer une base chiffrée
sqlcipher encrypted.db
```

```sql
PRAGMA key = 'votre-passphrase-forte';
CREATE TABLE secrets (id INTEGER PRIMARY KEY, token TEXT NOT NULL);
```

Sans la passphrase, le fichier est illisible, même avec un accès root au système de fichiers.

## Pour aller plus loin

Ce module couvre l'usage de SQLite dans son domaine de pertinence. Au-delà des limites listées dans [[SQLite 06 — Limites en production & SQLite vs PostgreSQL]] (accès réseau, concurrence d'écriture, haute disponibilité), la bascule vers un SGBD serveur s'impose — voir [[MySQL — Index des fiches]] pour l'alternative couverte dans ce vault.

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
