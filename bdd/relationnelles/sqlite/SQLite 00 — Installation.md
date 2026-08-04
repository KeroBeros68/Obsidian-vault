#bdd #sqlite #fondamentaux #installation

## Ce que SQLite n'est pas

Avant l'installation, poser les limites clairement :

- SQLite n'est pas un serveur de base de données — il n'écoute sur aucun port, n'accepte aucune connexion réseau. C'est une bibliothèque embarquée dans l'application.
- SQLite ne gère pas plusieurs écrivains simultanés — un seul processus écrit à la fois, les autres attendent ou échouent avec `SQLITE_BUSY`.
- SQLite n'a pas de système de rôles ni de permissions SQL — la sécurité repose sur les permissions du fichier.
- SQLite ne remplace pas PostgreSQL ni MySQL pour une application réseau multi-utilisateur avec forte concurrence d'écriture.

> [!info] Le moteur le plus déployé au monde
> SQLite est intégré dans Android, iOS, tous les navigateurs web, Python, PHP et des centaines d'autres logiciels — malgré (ou grâce à) ces limites volontaires. Le projet évolue activement : typage strict, JSON natif, colonnes générées.

## Deux choses distinctes portent le même nom

| | Rôle |
|---|------|
| Bibliothèque SQLite | Le moteur lui-même, présent sur pratiquement tout système, tiré par de nombreux paquets applicatifs |
| CLI `sqlite3` | L'outil en ligne de commande pour créer, inspecter et sauvegarder une base — c'est celui qui manque parfois |

## Installer la CLI

```bash
# Debian / Ubuntu — le paquet fournit la CLI, la bibliothèque est déjà tirée par le système
sudo apt update && sudo apt install -y sqlite3

# RHEL / Fedora / AlmaLinux — le paquet s'appelle simplement "sqlite"
sudo dnf install -y sqlite

# macOS — Homebrew donne accès aux fonctionnalités récentes (STRICT tables, sqlite3_rsync)
brew install sqlite
```

> [!tip] Souvent déjà installé
> La plupart des distributions Linux embarquent SQLite par défaut. `sqlite3 --version` suffit à vérifier.

## Vérifier la version

```bash
sqlite3 --version
```

```
3.46.1 2024-08-13 09:16:08 ...
```

> [!warning] Le numéro de version compte
> Plusieurs fonctionnalités (STRICT tables, colonnes générées, JSON, JSONB, `sqlite3_rsync`) ont un seuil d'apparition précis — voir [[SQLite 04 — Fonctionnalités modernes]] et [[SQLite 05 — Sauvegarde et restauration]]. Une version ancienne refuse la syntaxe correspondante avec une simple erreur de syntaxe, sans message explicite sur la cause réelle.

## Pour aller plus loin

La prise en main de la CLI `sqlite3` (créer une base, requêter, commandes dot) est couverte dans [[SQLite 01 — Premiers pas en CLI]].

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
