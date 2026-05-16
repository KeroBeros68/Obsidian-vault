#ia #langchain #embeddings #vectorstores #rag

## Embeddings et Vectorstores

## Embeddings — texte → vecteurs

```python
# OpenAI (payant, excellent)
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# HuggingFace (gratuit, open-source, supporte le français)
from langchain_community.embeddings import HuggingFaceEmbeddings
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)

# Tester manuellement
vecteur = embeddings.embed_query("Qu'est-ce que le RAG ?")
print(f"Dimension : {len(vecteur)}")   # → 384 dimensions
print(f"Extrait : {vecteur[:3]}")      # → [0.023, -0.187, 0.412]

# Embedder plusieurs textes en batch
vecteurs = embeddings.embed_documents(["texte 1", "texte 2", "texte 3"])
```

> [!tip] Pour le français
> `paraphrase-multilingual-MiniLM-L12-v2` est gratuit et fonctionne bien en français. `text-embedding-3-small` d'OpenAI est plus précis mais payant.

## Vectorstores — stocker et chercher

```python
from langchain_community.vectorstores import Chroma

# Créer depuis des chunks
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./db"   # sauvegarde sur disque
)

# Recharger depuis le disque
vectorstore = Chroma(
    persist_directory="./db",
    embedding_function=embeddings
)

# Recherche par similarité
résultats = vectorstore.similarity_search("retour produit", k=3)
for doc in résultats:
    print(doc.page_content[:100])

# Avec score de similarité (0=identique, 1=différent pour cosinus)
résultats_score = vectorstore.similarity_search_with_score("retour produit", k=3)
for doc, score in résultats_score:
    print(f"Score: {score:.3f} | {doc.page_content[:80]}")

# Ajouter des documents à un vectorstore existant
vectorstore.add_documents(nouveaux_chunks)

# Supprimer des documents
vectorstore.delete(ids=["id_doc_1", "id_doc_2"])
```

## Autres vectorstores populaires

```python
# FAISS — ultra-rapide, Meta (local)
from langchain_community.vectorstores import FAISS
vectorstore = FAISS.from_documents(chunks, embeddings)
vectorstore.save_local("./faiss_index")
vectorstore = FAISS.load_local("./faiss_index", embeddings)

# Qdrant — production, open-source
from langchain_community.vectorstores import Qdrant
vectorstore = Qdrant.from_documents(
    chunks, embeddings,
    url="http://localhost:6333",
    collection_name="ma_collection"
)

# Pinecone — cloud managé
from langchain_pinecone import PineconeVectorStore
vectorstore = PineconeVectorStore.from_documents(
    chunks, embeddings, index_name="mon-index"
)
```

## Comparaison des vectorstores

| Vectorstore | Type | Points forts |
|---|---|---|
| `Chroma` | Local | Simple, parfait pour débuter |
| `FAISS` | Local | Ultra-rapide sur les grandes collections |
| `Qdrant` | Local/Cloud | Filtres avancés, production |
| `Pinecone` | Cloud | Managé, scalable, facile |
| `pgvector` | PostgreSQL | Si tu as déjà Postgres |
