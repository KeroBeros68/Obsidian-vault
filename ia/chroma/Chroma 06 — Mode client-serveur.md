#ia #chroma #serveur #production #client-serveur #pratique

## Mode client-serveur

Le mode client-serveur permet à plusieurs processus ou machines de partager la même instance Chroma. Indispensable dès qu'on a plusieurs workers ou une séparation entre l'indexation et le serving.

## Pourquoi le mode serveur ?

```
Mode persistant local (PersistentClient) :
  → Un seul processus peut accéder à la DB à la fois
  → Parfait pour le développement solo

Mode client-serveur :
  → Plusieurs processus / machines partagent la même DB
  → Serveur web FastAPI + Chroma en backend
  → API REST standard → plusieurs langages peuvent l'utiliser
  → Indispensable pour : plusieurs workers, Docker, Kubernetes
```

## Lancer le serveur Chroma

```bash
# Installation du package si pas déjà fait
pip install chromadb

# Lancer le serveur avec persistance sur disque
chroma run --path ./db_chroma --port 8001

# Lancer en arrière-plan
chroma run --path ./db_chroma --port 8001 &

# Vérifier que le serveur tourne
curl http://localhost:8001/api/v2/heartbeat
# → {"nanosecond heartbeat": 1234567890}
```

## Se connecter au serveur depuis Python

```python
import chromadb

# Client HTTP — se connecte au serveur
client = chromadb.HttpClient(
    host="localhost",
    port=8001
)

# Vérifier la connexion
print(client.heartbeat())

# Utilisation identique au client local
collection = client.get_or_create_collection("ma_collection")
collection.add(ids=["1"], documents=["texte"], metadatas=[{}])
résultats = collection.query(query_texts=["texte"], n_results=1)
```

## Authentification (production)

```python
import chromadb
from chromadb.config import Settings

# Client avec auth basique
client = chromadb.HttpClient(
    host="mon-serveur.com",
    port=8001,
    settings=Settings(
        chroma_client_auth_provider="chromadb.auth.basic_authn.BasicAuthClientProvider",
        chroma_client_auth_credentials="admin:mot_de_passe_secret"
    )
)
```

## Docker — déploiement conteneurisé

```yaml
# docker-compose.yml
version: "3.9"

services:
  chroma:
    image: chromadb/chroma:latest
    ports:
      - "8001:8000"
    volumes:
      - ./chroma_data:/chroma/chroma   # persistance des données
    environment:
      - CHROMA_SERVER_AUTH_CREDENTIALS=admin:password
      - CHROMA_SERVER_AUTH_PROVIDER=chromadb.auth.basic_authn.BasicAuthenticationServerProvider
    restart: unless-stopped

  # Ton application qui utilise Chroma
  app:
    build: .
    depends_on:
      - chroma
    environment:
      - CHROMA_HOST=chroma
      - CHROMA_PORT=8000
```

```bash
docker-compose up -d

# L'app se connecte à Chroma via le réseau Docker
client = chromadb.HttpClient(host="chroma", port=8000)
```

## Intégration LangChain en mode serveur

```python
import chromadb
from langchain_chroma import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings

# Se connecter au serveur
chroma_client = chromadb.HttpClient(host="localhost", port=8001)

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)

# Initialiser LangChain Chroma depuis le client HTTP
vectorstore = Chroma(
    client=chroma_client,
    collection_name="docs_prod",
    embedding_function=embeddings
)

# Utilisation identique — aucun changement dans le reste du code
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})
```

## Architecture recommandée en production

```
┌─────────────────────────────────────────────────────┐
│                   PRODUCTION                         │
│                                                      │
│  ┌──────────────┐    ┌─────────────────────────────┐│
│  │  INDEXEUR    │    │      API SERVING             ││
│  │  (script     │    │  (FastAPI + LangChain)       ││
│  │   batch)     │    │  N workers                  ││
│  └──────┬───────┘    └──────────────┬──────────────┘│
│         │ HttpClient                │ HttpClient     │
│         └──────────────┬────────────┘               │
│                        ↓                             │
│              ┌─────────────────┐                    │
│              │  Chroma Server  │                    │
│              │  (Docker)       │                    │
│              │  port 8001      │                    │
│              └────────┬────────┘                    │
│                       │                             │
│              ┌────────┴────────┐                    │
│              │   Volume disque │                    │
│              │   (persistance) │                    │
│              └─────────────────┘                    │
└─────────────────────────────────────────────────────┘
```

## Séparer l'indexation du serving

```python
# ── Script d'indexation (tourne périodiquement) ───────────
# indexer.py
import chromadb
from langchain_chroma import Chroma

client = chromadb.HttpClient(host="chroma-server", port=8001)
vectorstore = Chroma(client=client, collection_name="docs", embedding_function=ef)

nouveaux_chunks = charger_nouveaux_documents()
vectorstore.add_documents(nouveaux_chunks)
print(f"Index mis à jour : {vectorstore._collection.count()} documents")


# ── API de serving (tourne en continu) ────────────────────
# serving.py
import chromadb
from langchain_chroma import Chroma
from fastapi import FastAPI

app = FastAPI()
client = chromadb.HttpClient(host="chroma-server", port=8001)
vectorstore = Chroma(client=client, collection_name="docs", embedding_function=ef)
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})

@app.get("/search")
def search(query: str):
    docs = retriever.invoke(query)
    return [{"texte": d.page_content, "source": d.metadata} for d in docs]
```

> [!tip] Séparer indexation et serving
> En production, sépare toujours le processus d'indexation du serving. L'indexation peut être longue et consommer beaucoup de ressources — elle ne doit pas bloquer les requêtes des utilisateurs.

> [!warning] Pas de PersistentClient avec plusieurs workers
> Si tu utilises `PersistentClient` (accès direct aux fichiers) avec plusieurs workers Uvicorn/Gunicorn, tu auras des corruptions de données. Utilise **toujours** `HttpClient` en mode multi-processus.
