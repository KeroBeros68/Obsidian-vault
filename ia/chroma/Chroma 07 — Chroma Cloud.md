#ia #chroma #cloud #saas #production

## Chroma Cloud

Chroma Cloud est l'offre SaaS hébergée de Chroma. Même API qu'en local, zéro infrastructure à gérer.

## Configuration

```bash
# S'authentifier via la CLI
pip install chromadb
chroma login
chroma connect  # configure CHROMA_TENANT, CHROMA_DATABASE, CHROMA_API_KEY
```

```python
import os
import chromadb

# Client Cloud
client = chromadb.CloudClient(
    tenant=os.getenv("CHROMA_TENANT"),
    database=os.getenv("CHROMA_DATABASE"),
    api_key=os.getenv("CHROMA_API_KEY")
)

# Utilisation identique au client local
collection = client.get_or_create_collection("ma_collection")
collection.add(ids=["1"], documents=["texte"], metadatas=[{}])
```

## Via LangChain

```python
from langchain_chroma import Chroma

vectorstore = Chroma(
    collection_name="docs_prod",
    embedding_function=embeddings,
    chroma_cloud_api_key=os.getenv("CHROMA_API_KEY"),
    tenant=os.getenv("CHROMA_TENANT"),
    database=os.getenv("CHROMA_DATABASE"),
)
# Aucun autre changement de code — même API qu'en local
```

## Local vs Serveur vs Cloud

| | Local (Persistent) | Serveur (HTTP) | Cloud |
|---|---|---|---|
| **Configuration** | Zéro | Docker / VM | Variables env |
| **Multi-processus** | ❌ | ✅ | ✅ |
| **Scalabilité** | Limitée | Selon VM | Automatique |
| **Backup** | Manuel | Manuel | Automatique |
| **Coût** | Gratuit | Infra | Payant |
| **Usage** | Dev / proto | Prod on-premise | Prod SaaS |

> [!tip] Commencer local, migrer vers Cloud
> Développe avec `PersistentClient`, déploie avec le serveur Docker, migre vers Chroma Cloud si tu as besoin de scalabilité automatique et de zéro maintenance.
