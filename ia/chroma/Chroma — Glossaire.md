#ia #chroma #glossaire #référence

| Terme | Définition |
|---|---|
| **ChromaDB** | Base de données vectorielle open-source (Apache 2.0). Stocke et recherche des embeddings. Simple, local, intégration LangChain native. |
| **Collection** | Unité de stockage dans Chroma — équivalent d'une table. Regroupe des documents avec leurs embeddings et métadonnées. |
| **PersistentClient** | Client Chroma avec stockage SQLite sur disque. Recommandé pour le développement et les petits projets. Un seul processus à la fois. |
| **HttpClient** | Client Chroma qui se connecte à un serveur Chroma via HTTP. Permet l'accès multi-processus et multi-machines. |
| **CloudClient** | Client Chroma pour Chroma Cloud (SaaS hébergé). Même API qu'en local. |
| **get_or_create_collection()** | Méthode idempotente qui crée la collection si elle n'existe pas, ou la charge si elle existe. À préférer à `create_collection()`. |
| **embedding_function** | Fonction passée à la collection pour calculer automatiquement les embeddings des documents et des requêtes. |
| **hnsw:space** | Paramètre metadata de collection qui définit la métrique de distance. Valeurs : `"cosine"` (défaut), `"l2"`, `"ip"`. |
| **cosine** | Métrique de distance par défaut dans Chroma. Distance de 0 (identique) à 2 (opposé). Recommandée pour le texte. |
| **l2** | Distance euclidienne. Sensible à la magnitude des vecteurs — normalise tes embeddings si tu l'utilises. |
| **ip** | Inner Product (produit scalaire). Pour les embeddings déjà normalisés. |
| **query()** | Méthode principale de recherche vectorielle dans Chroma. Prend `query_texts` ou `query_embeddings`, retourne les k documents les plus proches. |
| **n_results** | Paramètre de `query()` pour le nombre de résultats à retourner. |
| **where** | Paramètre de filtre sur les métadonnées dans `query()` et `get()`. Supporte `$eq`, `$ne`, `$gt`, `$in`, `$and`, `$or`... |
| **where_document** | Paramètre de filtre sur le contenu textuel. Supporte `$contains` et `$not_contains`. |
| **include** | Paramètre de `query()` et `get()` qui contrôle les champs retournés. Valeurs : `"documents"`, `"metadatas"`, `"distances"`, `"embeddings"`. |
| **distances** | Scores de distance retournés par `query()`. Avec cosine : plus proche de 0 = plus similaire. Convertir en similarité : `1 - distance/2`. |
| **upsert()** | Méthode CRUD qui crée si absent ou met à jour si présent. Plus robuste que `add()` en production. |
| **from_documents()** | Méthode LangChain qui crée un vectorstore Chroma depuis une liste de Documents LangChain. |
| **as_retriever()** | Méthode qui transforme le vectorstore Chroma en Retriever LangChain chaînable avec `|`. |
| **similarity_score_threshold** | Type de recherche qui ne retourne que les documents au-dessus d'un score de similarité minimum. |
| **MMR** | Maximum Marginal Relevance. Mode de recherche qui retourne des résultats pertinents ET diversifiés. Évite les résultats redondants. |
| **langchain-chroma** | Package Python séparé (depuis LangChain v0.2) pour l'intégration Chroma dans LangChain. `from langchain_chroma import Chroma`. |
| **chroma run** | Commande CLI pour lancer un serveur Chroma HTTP. `chroma run --path ./db --port 8001`. |
| **breaking changes v1.x** | ChromaDB a subi un rewrite majeur en v1.x avec nouveau backend DuckDB. Incompatible avec les versions < 0.7.0 sans migration manuelle. |
