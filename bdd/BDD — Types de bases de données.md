#bdd #fondamentaux

## Bases de données relationnelles (RDBMS)

Organisent les données en tables (lignes/colonnes), reliées par des clés étrangères — le modèle proposé par Edgar F. Codd en 1970 (voir [[BDD — Généralités]]).

| Exemple | Particularité |
|---------|-------------------|
| SQLite | Embarquée, sans serveur séparé — voir cas d'usage mobile plus bas |
| MySQL | Très répandu pour les applications web, simple à opérer |
| PostgreSQL | Robuste, fonctionnalités avancées (types géospatiaux, JSON natif) |
| Oracle Database | Environnements d'entreprise, support étendu |

**Avantages** : transactions ACID fiables, requêtes complexes et jointures via SQL (voir [[SQL — Index des fiches]]), outils d'administration matures.
**Limites** : scalabilité principalement verticale (augmenter la capacité d'une seule machine), moins adapté aux données non structurées.

## Bases de données NoSQL

Ne suivent pas le modèle relationnel ; conçues pour des données non structurées ou semi-structurées et une scalabilité horizontale. Se déclinent en plusieurs familles :

| Famille | Exemples | Structure de données |
|---------|----------|---------------------------|
| Orientée document | MongoDB, CouchDB | Documents JSON, structure flexible par document |
| Clé-valeur | Redis, DynamoDB | Paires clé-valeur simples, lectures/écritures très rapides |
| Colonnes | Apache Cassandra, HBase | Optimisées pour de gros volumes répartis sur plusieurs serveurs |
| Graphe | Neo4j, ArangoDB | Nœuds et relations, adaptée aux données fortement connectées (réseaux sociaux, recommandations) |

**Avantages** : scalabilité horizontale (ajout de serveurs plutôt qu'un serveur plus puissant), modèle de données flexible, performances élevées sur des opérations ciblées.
**Limites** : pas de langage de requête universel (chaque famille a le sien), transactions ACID souvent absentes ou limitées — voir le compromis BASE dans [[BDD — Généralités]].

## Bases de données en mémoire

Stockent l'intégralité des données en RAM plutôt que sur disque, pour des temps de réponse extrêmement rapides.

| Exemple | Usage typique |
|---------|-------------------|
| Redis | Cache, sessions, structures de données (listes, ensembles) en plus du clé-valeur simple |
| Memcached | Cache pur, pour des données fréquemment lues |

**Avantages** : latence minimale.
**Limites** : capacité bornée par la RAM disponible, persistance non garantie par défaut en cas de panne (des mécanismes de sauvegarde existent, mais ne sont pas le mode natif).

## Bases de données distribuées

Répartissent les données sur plusieurs serveurs, souvent dans le cloud, pour la haute disponibilité et la tolérance aux pannes.

| Exemple | Particularité |
|---------|-------------------|
| Apache Cassandra | Haute disponibilité, aucun point de défaillance unique |
| Google Cloud Spanner | Relationnel distribué, cohérence forte malgré la distribution |

**Avantages** : disponibilité et scalabilité horizontale massive.
**Limites** : complexité de gestion accrue, latence potentielle liée à la distribution géographique.

## Comment choisir

| Besoin | Famille à privilégier |
|--------|----------------------------|
| Transactions complexes, intégrité stricte (finance, comptabilité) | Relationnel (RDBMS) |
| Catalogue de données semi-structurées, évolutif | Document (MongoDB) |
| Cache, sessions, temps réel | Clé-valeur/mémoire (Redis) |
| Très gros volumes, écriture distribuée | Colonnes (Cassandra) |
| Relations complexes entre entités | Graphe (Neo4j) |
| Haute disponibilité multi-région | Distribué (Spanner, Cassandra) |

## Cas particuliers

> [!warning] NoSQL n'est pas un remplaçant universel du relationnel
> Le choix dépend de la structure réelle des données et du besoin de cohérence, pas d'une supériorité générale d'une famille sur l'autre — une application financière qui a besoin de transactions ACID strictes reste mieux servie par un RDBMS qu'une base orientée document, même si cette dernière est plus simple à faire évoluer.

> [!info] Certains moteurs combinent plusieurs modèles
> PostgreSQL supporte nativement des colonnes JSON (façon document) en plus de son modèle relationnel ; ArangoDB revendique explicitement le multi-modèle (graphe + document + clé-valeur) — la frontière entre familles n'est pas toujours stricte en pratique.
