#ia #chroma #pièges #erreurs #debugging

## 🪤 Piège 1 — IDs dupliqués avec add()

```python
# ❌ Ajouter un document avec un ID déjà existant → erreur
collection.add(ids=["doc_1"], documents=["texte original"])
collection.add(ids=["doc_1"], documents=["texte mis à jour"])
# → chromadb.errors.DuplicateIDError: doc_1 already exists

# ✅ Utiliser upsert() — crée ou met à jour silencieusement
collection.upsert(ids=["doc_1"], documents=["texte mis à jour"])
```

> [!tip] upsert() par défaut en production
> En production, utilise systématiquement `upsert()` à la place de `add()`. Plus robuste et idempotent — si le script d'indexation est relancé, il met à jour sans planter.

---

## 🪤 Piège 2 — IDs non stables (UUID aléatoires)

```python
# ❌ IDs aléatoires à chaque run → impossible de mettre à jour
import uuid
for chunk in chunks:
    collection.add(
        ids=[str(uuid.uuid4())],   # ID différent à chaque exécution !
        documents=[chunk.page_content]
    )
# Si le script est relancé → doublons dans la collection

# ✅ IDs dérivés du contenu → stables et reproductibles
import hashlib
for chunk in chunks:
    id_stable = hashlib.md5(
        f"{chunk.metadata.get('source','')}_{chunk.page_content[:100]}".encode()
    ).hexdigest()
    collection.upsert(
        ids=[id_stable],
        documents=[chunk.page_content],
        metadatas=[chunk.metadata]
    )
```

---

## 🪤 Piège 3 — PersistentClient avec plusieurs workers

```python
# ❌ Plusieurs processus Uvicorn avec PersistentClient → corruption
# uvicorn app:app --workers 4
# → 4 processus accèdent simultanément aux mêmes fichiers SQLite

import chromadb
client = chromadb.PersistentClient("./db")   # ← dangereux en multi-processus

# ✅ Toujours HttpClient en multi-processus
# Lancer d'abord le serveur : chroma run --path ./db --port 8001
client = chromadb.HttpClient(host="localhost", port=8001)
```

---

## 🪤 Piège 4 — Mauvaise métrique de distance

```python
# ❌ Utiliser L2 avec des embeddings non normalisés
collection = client.get_or_create_collection(
    "ma_col",
    metadata={"hnsw:space": "l2"}   # L2 sensible à la magnitude
)
# → Résultats biaisés si les embeddings ont des magnitudes très différentes

# ✅ Cosine pour le texte (insensible à la magnitude)
collection = client.get_or_create_collection(
    "ma_col",
    metadata={"hnsw:space": "cosine"}   # recommandé pour le texte
)

# ✅ Ou normaliser les embeddings si tu utilises L2
import numpy as np
embedding_normalisé = (embedding / np.linalg.norm(embedding)).tolist()
```

---

## 🪤 Piège 5 — Métadonnées avec types non supportés

```python
# ❌ Listes ou dicts dans les métadonnées → erreur
collection.add(
    ids=["1"],
    documents=["texte"],
    metadatas=[{
        "tags": ["python", "rag", "chroma"],  # ← liste non supportée
        "infos": {"auteur": "alice"}           # ← dict non supporté
    }]
)

# ✅ Seulement str, int, float, bool
collection.add(
    ids=["1"],
    documents=["texte"],
    metadatas=[{
        "tags": "python,rag,chroma",   # ← string séparée par virgules
        "auteur": "alice",             # ← string
        "nb_pages": 5,                 # ← int
        "publié": True                 # ← bool
    }]
)

# Pour filtrer les tags : where_document={"$contains": "python"}
# ou : where={"tags": {"$contains": "python"}} si Chroma le supporte
```

---

## 🪤 Piège 6 — Pas de seuil de similarité → réponses hors sujet

```python
# ❌ Retourner toujours k résultats même si aucun n'est pertinent
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})
# → Pour "recette de gateau au chocolat" dans une DB de docs produit
#   → retourne quand même 4 chunks qui n'ont rien à voir

# ✅ Ajouter un seuil de score
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"score_threshold": 0.7, "k": 4}
)
# → Retourne 0 résultats si rien n'est pertinent
# → Le prompt RAG peut alors répondre "je n'ai pas d'info sur ce sujet"
```

---

## 🪤 Piège 7 — Ne pas versionner chromadb dans requirements.txt

```
# ❌ Version non épinglée
chromadb

# → pip install chromadb peut installer la v1.x qui casse tout
#   si ton projet était sur la v0.x

# ✅ Épingler explicitement
chromadb==0.6.3
langchain-chroma==0.2.0
```

---

## 🪤 Piège 8 — Ancien import LangChain

```python
# ❌ Ancien import (déprécié depuis LangChain v0.2)
from langchain_community.vectorstores import Chroma

# ✅ Nouveau package dédié
from langchain_chroma import Chroma   # pip install langchain-chroma
```

---

## 🪤 Piège 9 — Ne pas sauvegarder avant une mise à jour majeure

```python
# ❌ Mettre à jour chromadb sans backup
# pip install -U chromadb
# → Si breaking changes → données potentiellement inaccessibles

# ✅ Toujours exporter avant de mettre à jour
data = collection.get(include=["documents", "embeddings", "metadatas"])
import json
with open("backup_avant_migration.json", "w") as f:
    json.dump({
        "ids": data["ids"],
        "documents": data["documents"],
        "embeddings": [e.tolist() for e in data["embeddings"]],
        "metadatas": data["metadatas"]
    }, f, ensure_ascii=False)
# Puis mettre à jour chromadb
# Puis réimporter depuis le JSON si nécessaire
```

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| IDs dupliqués | `upsert()` à la place de `add()` |
| IDs non stables | Hash MD5 du contenu comme ID |
| Multi-processus avec PersistentClient | `HttpClient` + serveur Chroma |
| Mauvaise métrique | `"cosine"` pour le texte |
| Types metadata non supportés | Seulement str, int, float, bool |
| Pas de seuil de similarité | `similarity_score_threshold` |
| Version non épinglée | `chromadb==x.y.z` dans requirements.txt |
| Ancien import LangChain | `from langchain_chroma import Chroma` |
| Pas de backup avant mise à jour | Exporter en JSON avant toute migration |
