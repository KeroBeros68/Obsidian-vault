#bdd #fondamentaux

## Qu'est-ce qu'un serveur de base de données ?

Un **serveur de base de données** est un logiciel qui stocke, organise et restitue des données de façon fiable, selon une architecture client-serveur : les clients (applications, scripts) envoient des requêtes, le serveur les exécute et retourne les résultats, en gérant lui-même transactions, cohérence et sécurité.

## Repères historiques

| Décennie | Évolution |
|----------|--------------|
| 1960 | Premiers SGBD, modèles hiérarchique (IBM IMS) et réseau (CODASYL) — complexes à naviguer |
| 1970 | Edgar F. Codd propose le **modèle relationnel** : données en tables, interrogées via SQL |
| 1980 | Essor des SGBD relationnels commerciaux (Oracle, DB2, SQL Server), transactions ACID |
| 2000 | Explosion des données web/mobile → émergence du **NoSQL** (MongoDB, Cassandra, Redis) pour la scalabilité |
| Aujourd'hui | Bases en mémoire (Redis), distribuées (Spanner), managées cloud (RDS), graphes (Neo4j) |

> [!info] Le modèle relationnel n'est qu'un modèle parmi d'autres
> Il reste dominant et le plus enseigné, mais les besoins de scalabilité horizontale et de données non structurées ont fait émerger des alternatives volontairement différentes plutôt que des successeurs — voir [[BDD — Types de bases de données]].

## Concepts clés, communs à la plupart des bases de données

- **Schéma** : la structure qui définit comment les données sont organisées — tables/colonnes/types en relationnel, structure plus flexible en NoSQL.
- **Requêtes** : les instructions pour interagir avec les données — SQL en relationnel, langages spécifiques à chaque moteur en NoSQL.
- **Index** : structures additionnelles qui accélèrent la recherche, au prix d'un coût d'écriture et d'espace disque supplémentaire.
- **Transaction** : un regroupement d'opérations qui doit être exécuté entièrement ou pas du tout, pour garantir la cohérence des données.

## ACID vs BASE : deux philosophies de cohérence

| | ACID (relationnel) | BASE (NoSQL distribué) |
|---|------------------------|----------------------------|
| **A**tomicité / **B**asic Availability | Chaque transaction est tout ou rien | Le système reste disponible même en cas de panne partielle |
| **C**ohérence / **S**oft state | La base passe d'un état valide à un autre | L'état peut évoluer dans le temps, même sans nouvelle écriture |
| **I**solation / **E**ventual consistency | Les transactions concurrentes ne s'interfèrent pas | Les données deviennent cohérentes à terme, pas immédiatement |
| **D**urabilité | Une transaction validée persiste, même après une panne | — |

> [!warning] BASE n'est pas "moins bien", c'est un compromis différent
> Un système distribué à grande échelle ne peut pas garantir simultanément une disponibilité totale et une cohérence stricte en cas de partition réseau (théorème CAP, non détaillé dans la ressource source mais sous-jacent à ce compromis). BASE choisit délibérément la disponibilité au prix d'une cohérence différée — un choix adapté à certains usages (réseaux sociaux, catalogues), inadapté à d'autres (comptabilité bancaire).

## Scalabilité : partitionnement, sharding, cache

- **Partitionnement** : diviser une base en segments plus petits, gérés indépendamment — améliore performance et gestion.
- **Sharding** : une forme de partitionnement où les segments sont répartis sur plusieurs machines physiques, pour la scalabilité horizontale.
- **Mise en cache** : stocker temporairement en mémoire les données fréquemment consultées, pour réduire le temps de réponse — voir [[BDD — Fonctionnalités des serveurs]].

## Cas particuliers

> [!tip] Un même moteur peut combiner plusieurs de ces mécanismes
> PostgreSQL, par exemple, reste un SGBD relationnel ACID par défaut, mais supporte le partitionnement de tables et peut être combiné à un cache externe (Redis) — ces concepts ne sont pas mutuellement exclusifs entre familles de bases de données.

## Pour aller plus loin

Les différents types de bases de données (relationnelles, NoSQL, mémoire, distribuées) sont détaillés dans [[BDD — Types de bases de données]] ; les fonctionnalités concrètes d'un serveur (transactions, réplication, sauvegarde...) dans [[BDD — Fonctionnalités des serveurs]] ; le vocabulaire complet dans [[BDD — Glossaire]]. La mise en pratique du langage de requête relationnel commence avec [[SQL — Index des fiches]].
