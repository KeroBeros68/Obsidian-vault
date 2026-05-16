#ia #chroma #collections #crud #pratique

## Collections et CRUD

Une collection est l'unité de stockage dans Chroma — équivalent d'une table. Toutes les opérations CRUD (Create, Read, Update, Delete) s'effectuent sur une collection.

## Gérer les collections

```python
import chromadb

client = chromadb.PersistentClient(path="./db")

# ── Créer ─────────────────────────────────────────────────
collection = client.create_collection(
    name="docs_techniques",
    metadata={
        "hnsw:space": "cosine",       # métrique de distance
        "description": "Docs produit" # metadata custom
    }
)

# ── Créer ou charger (idempotent — recommandé) ────────────
collection = client.get_or_create_collection("docs_techniques")

# ── Charger une collection existante ─────────────────────
collection = client.get_collection("docs_techniques")

# ── Lister toutes les collections ────────────────────────
collections = client.list_collections()
for col in collections:
    print(f"{col.name} : {col.count()} docs")

# ── Supprimer une collection ──────────────────────────────
client.delete_collection("docs_techniques")

# ── Renommer / modifier les métadonnées ───────────────────
collection.modify(name="docs_produit_v2")
```

## ADD — ajouter des documents

```python
# Ajout avec calcul automatique des embeddings
collection.add(
    ids=["001", "002", "003"],           # obligatoire, unique par collection
    documents=["texte 1", "texte 2", "texte 3"],
    metadatas=[
        {"source": "doc_a.pdf", "page": 1, "langue": "fr"},
        {"source": "doc_a.pdf", "page": 2, "langue": "fr"},
        {"source": "doc_b.md",  "page": 1, "langue": "en"},
    ]
)

# Ajout avec embeddings pré-calculés (sans embedding_function)
import numpy as np
embeddings_précalculés = np.random.rand(3, 384).tolist()

collection.add(
    ids=["004", "005", "006"],
    embeddings=embeddings_précalculés,   # vecteurs déjà calculés
    documents=["texte 4", "texte 5", "texte 6"],
    metadatas=[{}, {}, {}]
)

# Ajout en batch (automatiquement géré par Chroma)
# Chroma gère les grands volumes — pas besoin de batcher manuellement
ids_batch       = [f"chunk_{i}" for i in range(1000)]
documents_batch = [f"Contenu du chunk {i}" for i in range(1000)]
metadatas_batch = [{"index": i} for i in range(1000)]

collection.add(ids=ids_batch, documents=documents_batch, metadatas=metadatas_batch)
```

## GET — récupérer des documents

```python
# Par IDs spécifiques
résultat = collection.get(ids=["001", "003"])
print(résultat["documents"])   # → ["texte 1", "texte 3"]
print(résultat["metadatas"])   # → [{...}, {...}]

# Avec filtre sur les métadonnées
résultat = collection.get(
    where={"source": "doc_a.pdf"},           # filtre exact
    include=["documents", "metadatas"]       # champs à retourner
)

# Avec pagination
résultat = collection.get(
    limit=10,    # max 10 résultats
    offset=20,   # commencer au 21ème
    include=["documents", "metadatas", "embeddings"]
)

# Tout récupérer (attention sur les grands corpus)
tout = collection.get()
print(f"Total : {collection.count()} documents")
```

## UPDATE — mettre à jour des documents

```python
# Mettre à jour le texte et les métadonnées (recalcule les embeddings)
collection.update(
    ids=["001"],
    documents=["Texte mis à jour pour le document 001"],
    metadatas=[{"source": "doc_a.pdf", "page": 1, "version": 2}]
)

# UPSERT — créer si absent, mettre à jour si présent
collection.upsert(
    ids=["001", "999"],                          # 001 existe, 999 non
    documents=["Texte mis à jour", "Nouveau texte"],
    metadatas=[{"version": 3}, {"source": "nouveau.pdf"}]
)
# → 001 est mis à jour, 999 est créé
```

## DELETE — supprimer des documents

```python
# Par IDs
collection.delete(ids=["001", "002"])

# Par filtre sur les métadonnées
collection.delete(
    where={"source": "doc_a.pdf"}   # supprime tout ce qui vient de doc_a.pdf
)

# Par filtre ET contenu du texte
collection.delete(
    where={"langue": "en"},
    where_document={"$contains": "deprecated"}   # texte contient "deprecated"
)
```

## Métriques de distance disponibles

```python
# Cosine similarity (défaut, recommandé pour le texte)
collection = client.get_or_create_collection(
    name="ma_col",
    metadata={"hnsw:space": "cosine"}
)
# Distance cosine : 0 = identiques, 2 = opposés

# L2 (distance euclidienne)
collection = client.get_or_create_collection(
    name="ma_col",
    metadata={"hnsw:space": "l2"}
)

# IP (produit scalaire — pour embeddings normalisés)
collection = client.get_or_create_collection(
    name="ma_col",
    metadata={"hnsw:space": "ip"}
)
```

> [!tip] upsert() plutôt que add()
> En production, utilise `upsert()` plutôt que `add()`. Si un document avec le même ID est ajouté deux fois, `add()` lève une erreur, `upsert()` met à jour silencieusement. Beaucoup plus robuste dans les pipelines d'indexation.

> [!warning] IDs uniques et stables
> Les IDs doivent être uniques dans une collection et stables dans le temps. Utilise un hash du contenu (`hashlib.md5(texte.encode()).hexdigest()`) ou un identifiant métier stable. Ne jamais utiliser des UUIDs aléatoires générés à chaque run — ils rendent les mises à jour impossibles.
