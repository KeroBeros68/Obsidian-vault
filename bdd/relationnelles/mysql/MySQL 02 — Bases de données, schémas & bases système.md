#bdd #mysql #fondamentaux

## Base de données = schéma, dans MySQL

Une instance MySQL contient une ou plusieurs bases de données. Contrairement à PostgreSQL, où une base peut contenir plusieurs schémas, **MySQL traite les deux termes comme strictement synonymes** : un schéma = une base de données = un répertoire dans le `datadir`.

```sql
-- Ces deux commandes sont strictement identiques
CREATE DATABASE labdb;
CREATE SCHEMA labdb;
```

```bash
ls /var/lib/mysql/
# labdb/  mysql/  performance_schema/  sys/  ibdata1  binlog.000001  ...
```

Chaque base correspond à un répertoire ; en créant `labdb`, un dossier `labdb/` apparaît avec un fichier `.ibd` par table.

## Les quatre bases système

Sur une installation fraîche :

```sql
SHOW DATABASES;
-- information_schema, mysql, performance_schema, sys
```

| Base | Rôle |
|------|------|
| `mysql` | Comptes utilisateurs, privilèges, événements planifiés, métadonnées internes — équivalent du catalogue système de PostgreSQL |
| `information_schema` | Vue en lecture seule sur les métadonnées de toutes les bases (tables, colonnes, index, contraintes) — utile pour l'introspection |
| `performance_schema` | Métriques en temps réel (requêtes, verrous, I/O, threads, mémoire) — système d'instrumentation interne |
| `sys` | Vues simplifiées combinant `performance_schema` et `information_schema` pour faciliter le diagnostic |

> [!warning] Ne jamais modifier la base `mysql` directement
> Les tables d'authentification (`mysql.user`, `mysql.db`, `mysql.global_grants`) ne doivent jamais recevoir de `INSERT`/`UPDATE` directs — utiliser exclusivement `CREATE USER`, `GRANT` et `REVOKE`. Une modification directe peut corrompre le cache des privilèges et créer des incohérences que ces commandes officielles évitent.

## Cas particuliers

> [!info] `sys` n'ajoute aucune donnée, seulement de la lisibilité
> Tout ce qu'expose `sys` existe déjà dans `performance_schema`/`information_schema` — c'est une couche de vues pré-construites, pensée pour un diagnostic rapide sans écrire de requêtes complexes sur les tables brutes.

## Pour aller plus loin

Le détail des couches internes qui traitent une requête SQL (connecteur, parseur, optimiseur...) est dans [[MySQL 03 — Architecture interne (les couches de mysqld)]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
