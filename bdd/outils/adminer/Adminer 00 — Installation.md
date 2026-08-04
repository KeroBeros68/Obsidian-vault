#bdd #adminer #fondamentaux #installation

## Ce qu'est Adminer

Adminer (anciennement phpMinAdmin) est un outil d'administration de bases de données écrit en PHP, distribué sous la forme d'un **fichier unique** — pas d'installation, pas de dépendances à gérer, on dépose le fichier sur un serveur web et on s'y connecte depuis un navigateur.

| | Détail |
|---|--------|
| Moteurs pris en charge nativement | MySQL, MariaDB, PostgreSQL, CockroachDB, SQLite, MS SQL, Oracle |
| Moteurs via plugin | Elasticsearch, MongoDB, Redis, Firebird, ClickHouse, SimpleDB |
| Poids | ~487 Ko (version complète), ~332 Ko (version anglais uniquement) |
| Licence | Apache 2.0 / GPL 2 (au choix) |
| Prérequis | PHP 7.4+ (sources), PHP 5.3+ (fichier compilé) |

> [!info] Un seul fichier, un seul point d'entrée
> Contrairement à phpMyAdmin (des dizaines de fichiers, une configuration à maintenir), Adminer tient dans un `index.php` unique — ce qui simplifie le déploiement mais aussi la suppression une fois l'usage terminé, un point important pour la sécurité (voir [[Adminer 03 — Sécurisation]]).

## Installer le fichier unique

Télécharger la dernière version stable depuis le site officiel :

```bash
wget https://www.adminer.org/latest.php -O adminer.php
```

Variantes disponibles :

| Variante | URL | Usage |
|----------|-----|-------|
| Complète, multilingue | `latest.php` | Défaut, tous moteurs et langues |
| MySQL uniquement | `latest-mysql.php` | Fichier plus léger, un seul driver chargé |
| Anglais uniquement | `latest-en.php` | Fichier plus léger, une seule langue |

Placer `adminer.php` dans le répertoire servi par le serveur web (ex. `/var/www/html/`), puis ouvrir `http://serveur/adminer.php` dans un navigateur.

> [!warning] Ne jamais garder le nom `adminer.php` en production
> Ce nom est le premier scanné par les bots qui recherchent des instances Adminer exposées. Le renommer fait partie des mesures de base — voir [[Adminer 03 — Sécurisation]].

## Installer via Docker

Image officielle maintenue sur Docker Hub, port par défaut `8080` :

```bash
docker run -d -p 8080:8080 --name adminer adminer
```

```yaml
# docker-compose.yml
services:
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
  db:
    image: mysql:8.4
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
```

```bash
docker compose up -d
# http://localhost:8080
```

L'image embarque tous les plugins officiels et accepte des variables d'environnement de personnalisation — voir [[Adminer 05 — Plugins & personnalisation]].

## Installer via un gestionnaire de paquets

Des paquets communautaires existent pour Debian (`apt install adminer`), et des intégrations pour WordPress, Drupal, Omeka S. Ces paquets suivent le rythme de la distribution, pas toujours la dernière version — pour une instance à jour côté sécurité, préférer le téléchargement direct depuis `adminer.org`.

## Pour aller plus loin

La connexion et la navigation dans l'interface sont couvertes dans [[Adminer 01 — Prise en main]]. Avant toute exposition réseau, même en interne, lire impérativement [[Adminer 03 — Sécurisation]].

Sources : [Adminer — site officiel](https://www.adminer.org/en/), [Adminer sur GitHub](https://github.com/vrana/adminer/), [Adminer — Docker Hub (image officielle)](https://hub.docker.com/_/adminer/)
