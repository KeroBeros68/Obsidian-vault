#bdd #sqlite #avancé #limites #postgresql

## Limites en production

SQLite fonctionne très bien en mono-hôte, mais n'est pas fait pour tous les cas :

| Limite | Détail |
|--------|--------|
| Un seul écrivain à la fois | Même avec WAL, les écritures sont sérialisées — une application web à 50 écritures/seconde sature rapidement |
| Pas d'accès réseau | SQLite n'écoute sur aucun port, impossible de s'y connecter depuis un autre serveur |
| Pas de rôles ni de permissions SQL | Les droits dépendent uniquement des permissions fichier de l'OS |
| Pas de réplication intégrée | `sqlite3_rsync` est un outil externe de synchronisation, pas de la réplication temps réel |
| Pas de haute disponibilité | Pas de failover automatique, pas de cluster |
| Pas de partitionnement | Pas de sharding ni de partitionnement natif des tables |
| Taille pratique | Format jusqu'à 281 To en théorie, mais performances dégradées au-delà de ~1 To |
| Pas de supervision serveur | Pas de `pg_stat_activity`, pas de slow query log intégré, pas de métriques exposables |

SQLite n'apporte pas ce qu'on attend d'un SGBD serveur : gestion réseau, rôles, HA, réplication, supervision — c'est le prix de la simplicité, un bon compromis tant que le cas d'usage reste local et mono-instance. Voir [[MySQL — Index des fiches]] et [[BDD — Types de bases de données]] pour le point de comparaison côté SGBD client-serveur.

## SQLite vs PostgreSQL : quand migrer ?

Ce n'est pas une comparaison de performances — les deux moteurs ne jouent pas le même rôle. La question n'est pas « lequel est le meilleur » mais « le besoin dépasse-t-il le mono-hôte ». L'accès réseau à lui seul suffit souvent à trancher.

| Critère | SQLite | PostgreSQL |
|---------|--------|------------|
| Déploiement | Un fichier, zéro config | Serveur à installer et maintenir |
| Accès réseau | Local uniquement | Multi-clients réseau |
| Concurrence d'écriture | Un seul écrivain | Plusieurs écrivains simultanés (MVCC) |
| Permissions | Fichier OS | Rôles, `GRANT`, Row-Level Security |
| Réplication | `sqlite3_rsync` (externe, incrémental) | Streaming replication intégrée |
| Haute disponibilité | Non | Patroni, pgBouncer, failover automatique |
| Extensions | Limitées | PostGIS, `pg_stat_statements`, TimescaleDB... |
| JSON | `json_extract()`, JSONB | `jsonb`, index GIN, opérateurs avancés |
| Taille de la base | Confortable jusqu'à ~1 To | Plusieurs To sans problème |
| Backup | `.backup`, `VACUUM INTO`, `sqlite3_rsync` | `pg_dump`, PITR, WAL archiving |
| Supervision | Aucune intégrée | `pg_stat_activity`, slow log, exporters Prometheus |

### Règle de décision simple

Un seul critère de la seconde liste suffit à justifier PostgreSQL — pas besoin de les réunir tous. À l'inverse, tant que les cinq points de la première liste restent vrais, installer un serveur de base de données n'apporte rien d'autre qu'un composant supplémentaire à exploiter et à sauvegarder.

**Rester sur SQLite si :**

- la base est locale (même machine que l'application)
- un seul processus écrit à la fois
- l'application est mono-instance
- la taille reste raisonnable (< 100 Go)
- pas besoin de droits fins ou de supervision

**Passer à PostgreSQL si :**

- plusieurs utilisateurs ou services accèdent à la base via le réseau
- concurrence d'écriture significative
- besoin de permissions par rôle
- haute disponibilité ou réplication prévue
- la base va croître au-delà de ce qu'un fichier local gère confortablement

> [!tip] En cas de doute
> Si l'application est déjà multi-serveur ou le deviendra bientôt, partir directement sur PostgreSQL. Migrer une base SQLite vers PostgreSQL fonctionne (dump SQL + adaptations), mais il est toujours plus simple de partir sur la bonne architecture dès le début.

## Pour aller plus loin

L'administration au quotidien d'une base SQLite qui reste dans son domaine de pertinence (permissions fichier, intégrité, chiffrement) est couverte dans [[SQLite 07 — Bonnes pratiques admin]].

Sources : [SQLite : la base de données embarquée pour admins et DevOps — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/)
