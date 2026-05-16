#ia #chroma #migration #backup #production

## Migration et backup

ChromaDB a eu des breaking changes majeurs entre versions (notamment la v1.x). Avoir une stratégie de backup et de migration est indispensable en production.

## Export — sauvegarder les données

```python
import json
from langchain_chroma import Chroma

# Méthode universelle — export en JSON (indépendant du backend)
def exporter_collection(vectorstore: Chroma, chemin: str):
    """Exporte une collection Chroma en JSON portable."""
    data = vectorstore._collection.get(
        include=["documents", "embeddings", "metadatas"]
    )

    export = {
        "ids":        data["ids"],
        "documents":  data["documents"],
        "embeddings": [e.tolist() if hasattr(e, 'tolist') else e
                       for e in data["embeddings"]],
        "metadatas":  data["metadatas"]
    }

    with open(chemin, "w", encoding="utf-8") as f:
        json.dump(export, f, ensure_ascii=False, indent=2)

    print(f"Export : {len(export['ids'])} documents → {chemin}")

exporter_collection(vectorstore, "./backup_chroma.json")
```

## Import — restaurer les données

```python
import json
import chromadb
from langchain_chroma import Chroma

def importer_collection(
    chemin: str,
    client: chromadb.ClientAPI,
    collection_name: str,
    embedding_function=None
) -> Chroma:
    """Importe une collection depuis un JSON exporté."""

    with open(chemin, "r", encoding="utf-8") as f:
        data = json.load(f)

    collection = client.get_or_create_collection(
        collection_name,
        embedding_function=embedding_function
    )

    # Importer par batches pour éviter les timeouts
    batch_size = 100
    total = len(data["ids"])

    for i in range(0, total, batch_size):
        batch_ids        = data["ids"][i:i+batch_size]
        batch_documents  = data["documents"][i:i+batch_size]
        batch_embeddings = data["embeddings"][i:i+batch_size]
        batch_metadatas  = data["metadatas"][i:i+batch_size]

        collection.upsert(
            ids=batch_ids,
            documents=batch_documents,
            embeddings=batch_embeddings,   # réutilise les embeddings exportés
            metadatas=batch_metadatas
        )
        print(f"Importé {min(i+batch_size, total)}/{total}")

    print(f"Import terminé : {collection.count()} documents")
    return Chroma(client=client, collection_name=collection_name,
                  embedding_function=embedding_function)

# Restaurer
client_nouveau = chromadb.PersistentClient("./db_nouveau")
vectorstore = importer_collection(
    "./backup_chroma.json",
    client_nouveau,
    "docs_restaurés"
)
```

## Migration vers une nouvelle version de Chroma

```python
# Stratégie recommandée lors d'une mise à jour majeure
# (notamment la migration vers chromadb >= 1.0)

# 1. Exporter depuis l'ancienne version
#    (faire tourner ce code avec l'ancienne version installée)
exporter_collection(ancien_vectorstore, "./migration_export.json")

# 2. Mettre à jour chromadb
#    pip install -U chromadb

# 3. Réimporter dans la nouvelle version
client_nouveau = chromadb.PersistentClient("./db_v2")
importer_collection("./migration_export.json", client_nouveau, "ma_collection")

# Note : si les embeddings sont dans l'export, pas besoin de les recalculer
# → économie massive de temps et de coût API
```

## Backup automatique

```python
import shutil
from datetime import datetime
from pathlib import Path

def backup_chroma(db_path: str, backup_dir: str = "./backups"):
    """Copie le dossier Chroma avec timestamp."""
    Path(backup_dir).mkdir(exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    dest = f"{backup_dir}/chroma_backup_{timestamp}"
    shutil.copytree(db_path, dest)
    print(f"Backup créé : {dest}")

# Planifier via cron (Linux/Mac)
# 0 2 * * * python -c "from backup import backup_chroma; backup_chroma('./db')"

# Ou dans le code, avant chaque mise à jour majeure
backup_chroma("./db_chroma")
```

## Épingler la version Chroma

```txt
# requirements.txt
chromadb==0.6.3     # ← épingler explicitement
langchain-chroma==0.2.0
```

> [!warning] Ne jamais mettre à jour chromadb sans tester
> Les versions majeures de Chroma ont causé des incompatibilités sévères (nouveau backend DuckDB en v1.x, API changée). Toujours tester sur un environnement de staging avant de mettre à jour en production.

> [!tip] JSON comme format de migration universel
> Le JSON ne dépend d'aucune version de Chroma. C'est le format le plus sûr pour les backups et migrations. SQLite, Parquet, DuckDB — tous ces backends peuvent changer. Le JSON reste stable.
