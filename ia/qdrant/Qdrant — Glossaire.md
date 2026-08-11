#ia #qdrant #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Qdrant** | Base vectorielle serveur écrite en Rust, s'exécutant comme un service indépendant auquel les applications se connectent par le réseau. |
| **Collection** | Unité de stockage de Qdrant, l'équivalent d'une table — typée à sa création par la dimension des vecteurs et la distance de comparaison. |
| **Point** | Unité indexée dans Qdrant, réunissant un identifiant, un vecteur, et un payload. |
| **Payload** | Dictionnaire libre de métadonnées attaché à un point (texte d'origine, sujet, année...) — filtrable à la recherche, l'atout distinctif de Qdrant. |
| **`upsert`** | Opération insérant un point, ou le mettant à jour s'il existe déjà (*update* + *insert*). |
| **`query_points`** | Méthode de recherche sémantique, renvoyant les points classés par similarité décroissante avec leur score et leur payload. |
| **`Filter` / `FieldCondition`** | Construction d'un filtre sur le payload, restreignant la recherche sémantique à un sous-ensemble de points avant même le calcul de similarité. |
| **`must` / `should` / `must_not`** | Clauses d'un `Filter` : toutes les conditions vraies, au moins une vraie, ou aucune vraie. |
| **Multi-tenant** | Plusieurs équipes ou clients partageant une seule collection, chaque recherche restant cloisonnée à son périmètre par un filtre systématique sur le payload. |
