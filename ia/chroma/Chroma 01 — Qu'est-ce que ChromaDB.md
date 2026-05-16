#ia #chroma #bases #définition #vectorstore

## Qu'est-ce que ChromaDB ?

ChromaDB est une **base de données vectorielle open-source** conçue pour stocker, rechercher et filtrer des embeddings. C'est le vectorstore de référence pour débuter avec le RAG — zéro configuration, zéro credentials, tout local.

## Pourquoi Chroma ?

```
Elasticsearch / Pinecone / Qdrant :
  → Nécessite un serveur, des credentials, une infra
  → Complexe à configurer pour débuter

ChromaDB :
  → pip install chromadb → c'est prêt
  → Stockage local SQLite (persistant) ou en RAM (éphémère)
  → API Python simple et intuitive
  → Intégration native LangChain et LlamaIndex
  → Passage au mode client-serveur quand besoin
```

## Les 4 modes de déploiement

```
1. In-Memory (éphémère)
   client = chromadb.Client()
   → Données perdues à la fin du programme
   → Idéal pour les tests unitaires

2. Persistant local (SQLite)
   client = chromadb.PersistentClient(path="./db")
   → Données sauvegardées sur disque
   → Idéal pour le développement et les petits projets

3. Client-Serveur (HTTP)
   server : chroma run --path ./db
   client : chromadb.HttpClient(host="localhost", port=8001)
   → Plusieurs processus / machines partagent la même DB
   → Idéal pour la production locale ou les équipes

4. Chroma Cloud (SaaS)
   client = chromadb.CloudClient(tenant=..., database=..., api_key=...)
   → Hébergé, managé, scalable
   → Idéal pour la production sans infra à gérer
```

## Concepts clés

```
Client        : connexion à Chroma (in-memory, persistant, HTTP, cloud)
Collection    : équivalent d'une table — regroupe des documents similaires
Document      : texte brut stocké dans Chroma
Embedding     : vecteur numérique associé au document
Metadata      : dict de métadonnées filtrable (source, date, catégorie...)
ID            : identifiant unique de chaque document dans une collection
```

## Ce que Chroma stocke pour chaque entrée

```python
# Chaque entrée dans Chroma contient :
{
    "id":        "doc_001",                    # identifiant unique (obligatoire)
    "document":  "Texte brut du chunk...",     # contenu textuel
    "embedding": [0.23, -0.87, 0.41, ...],    # vecteur (384 à 1536 dims)
    "metadata":  {"source": "rapport.pdf",    # dict filtrable
                  "page": 3,
                  "date": "2025-01-15"}
}
```

## Positionnement parmi les vectorstores

| Vectorstore | Type | Points forts | Limite |
|---|---|---|---|
| **Chroma** | Local / Cloud | Simple, zéro config, LangChain natif | Moins scalable que Qdrant/Pinecone |
| **FAISS** | Local | Ultra-rapide, Meta | Pas de filtres metadata, pas de persistance native |
| **Qdrant** | Local / Cloud | Filtres avancés, très performant | Plus complexe à configurer |
| **Pinecone** | Cloud | Managé, scalable | Payant au-delà du free tier |
| **pgvector** | PostgreSQL | Si déjà Postgres | Moins optimisé que les DB dédiées |

> [!tip] Chroma pour débuter, Qdrant pour la production
> Chroma est le meilleur point d'entrée — simple, local, bien documenté. Migre vers Qdrant ou Pinecone quand tu as besoin de filtres avancés, de haute disponibilité ou de millions de vecteurs.

> [!warning] Chroma 1.x — breaking changes
> ChromaDB a subi un rewrite majeur en version 1.x (nouveau backend DuckDB). Les projets sur chromadb < 0.7.0 nécessitent une migration manuelle. Toujours épingler la version dans ton `requirements.txt`.
