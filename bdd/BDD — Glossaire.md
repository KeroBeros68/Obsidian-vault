#bdd #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Serveur de base de données** | Logiciel qui stocke, organise et restitue des données selon une architecture client-serveur, en gérant transactions, cohérence et sécurité. |
| **Modèle relationnel** | Modèle proposé par Edgar F. Codd (1970) organisant les données en tables composées de lignes et de colonnes, reliées par des clés. |
| **Schéma** | Structure qui définit comment les données sont organisées — tables/colonnes/types en relationnel, structure plus flexible en NoSQL. |
| **Index (BDD)** | Structure additionnelle accélérant la recherche sur une colonne, au prix d'un coût d'écriture et d'espace disque supplémentaires. |
| **Transaction** | Regroupement de plusieurs opérations en une seule unité, appliquée entièrement ou pas du tout. |
| **ACID** | Atomicité, Cohérence, Isolation, Durabilité — propriétés garantissant la fiabilité des transactions dans une base relationnelle. |
| **BASE** | Basic Availability, Soft state, Eventual consistency — modèle de cohérence plus souple, utilisé par les bases NoSQL distribuées à la place d'ACID. |
| **Partitionnement** | Division d'une base de données en segments plus petits, gérés indépendamment pour améliorer performance et gestion. |
| **Sharding** | Forme de partitionnement où les segments sont répartis sur plusieurs machines physiques, pour la scalabilité horizontale. |
| **RDBMS** | *Relational Database Management System* — SGBD relationnel (PostgreSQL, MySQL, Oracle...), organisant les données en tables reliées. |
| **NoSQL** | Famille de bases de données ne suivant pas le modèle relationnel, conçues pour des données non structurées et une scalabilité horizontale. |
| **Base orientée document** | Base NoSQL stockant les données sous forme de documents (souvent JSON) à structure flexible, ex. MongoDB. |
| **Base clé-valeur** | Base NoSQL stockant des paires clé-valeur simples, optimisée pour des lectures/écritures très rapides, ex. Redis. |
| **Base en colonnes** | Base NoSQL optimisée pour de gros volumes de données réparties sur plusieurs serveurs, ex. Apache Cassandra. |
| **Base graphe** | Base NoSQL représentant les données comme des nœuds et des relations, adaptée aux données fortement connectées, ex. Neo4j. |
| **Base en mémoire** | Base stockant l'intégralité des données en RAM pour un temps de réponse minimal (Redis, Memcached), au prix d'une capacité bornée par la mémoire disponible. |
| **Base distribuée** | Base répartissant les données sur plusieurs serveurs pour la haute disponibilité et la tolérance aux pannes (Cassandra, Google Cloud Spanner). |
| **Réplication** | Copie des données d'un serveur vers un ou plusieurs autres, pour la redondance et la haute disponibilité (master-slave ou multi-master). |
| **`pg_dump` / `pg_restore`** | Outils PostgreSQL de sauvegarde et de restauration d'une base de données. |
| **Journal des transactions (transaction log)** | Enregistrement de chaque modification apportée aux données, utilisé pour la récupération après panne et pour garantir la durabilité. |
| **Haute disponibilité** | Capacité d'un système à rester opérationnel malgré la défaillance d'un serveur, combinant réplication, clustering et bascule automatique. |
