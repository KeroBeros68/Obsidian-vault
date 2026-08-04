#bdd #mariadb #installation #fondamentaux

## Deux sources de paquets

MariaDB peut s'installer de deux façons sur Debian/Ubuntu : depuis le dépôt Debian par défaut (`mariadb-server` déjà présent dans les dépôts officiels de la distribution), ou depuis le dépôt officiel de MariaDB Corporation, qui propose des versions plus récentes et un choix de série précis.

> [!info] Pourquoi deux dépôts existent
> Debian gèle une version de MariaDB au moment de la sortie de chaque release (ex. MariaDB 10.11 dans Debian 12). Le dépôt MariaDB Corporation permet d'installer une série plus récente (11.8, 12.3...) sans attendre la prochaine version de la distribution.

## Installer depuis le dépôt officiel MariaDB

```bash
curl -LsS -O https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
sudo bash mariadb_repo_setup --mariadb-server-version=11.8

sudo apt update
sudo apt install mariadb-server mariadb-client
```

`mariadb-server` fournit le service (`mariadbd`, voir [[MariaDB 01 — Histoire du fork, licence & compatibilité MySQL]]), `mariadb-client` fournit le client CLI (`mariadb`) pour l'administration locale, les connexions distantes et les dumps.

## Installer depuis le dépôt Debian par défaut

```bash
sudo apt update
sudo apt install mariadb-server
```

Plus simple, mais fige la version à celle packagée par Debian pour la release en cours (ex. 10.11 sur Debian 12) — pas de dépôt ni de clé GPG supplémentaire à gérer.

## Vérifier l'installation

```bash
sudo systemctl status mariadb
mariadb --version
```

```sql
SELECT VERSION();
-- 11.8.6-MariaDB
```

> [!info] Le binaire client s'appelle `mariadb`, pas `mysql`
> Depuis MariaDB 10.4+, le client en ligne de commande officiel est `mariadb` (l'ancien nom `mysql` reste disponible comme lien symbolique pour compatibilité). Les deux commandes sont strictement équivalentes.

## mariadb-secure-installation

Équivalent du `mysql_secure_installation` de MySQL (voir [[MySQL 00 — Installation]]) : supprime les comptes anonymes, restreint l'accès root réseau, retire la base `test`.

```bash
sudo mariadb-secure-installation
```

> [!warning] root sans mot de passe n'est pas un oubli
> Par défaut, `root@localhost` s'authentifie via le plugin `unix_socket` (identité du système d'exploitation), pas via un mot de passe MariaDB — voir [[MariaDB 08 — Authentification (unix_socket, ed25519 & mysql_native_password)]]. C'est une différence structurelle avec MySQL, qui utilise un mot de passe généré ou vide selon le paquet.

## Pour aller plus loin

L'histoire du fork, sa licence et sa compatibilité avec MySQL sont détaillées dans [[MariaDB 01 — Histoire du fork, licence & compatibilité MySQL]].

Sources : [MariaDB Package Repository Setup and Usage — MariaDB Documentation](https://mariadb.com/docs/server/server-management/install-and-upgrade-mariadb/installing-mariadb/binary-packages/mariadb-package-repository-setup-and-usage)
