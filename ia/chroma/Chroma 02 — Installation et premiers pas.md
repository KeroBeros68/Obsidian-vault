#ia #chroma #installation #bases #pratique

## Installation et premiers pas

## Installation

```bash
# ChromaDB core
pip install chromadb

# Intégration LangChain (package séparé depuis langchain v0.2)
pip install langchain-chroma

# Embeddings locaux gratuits
pip install sentence-transformers
```

## Les 3 types de clients

```python
import chromadb

# ── 1. In-Memory — éphémère, pour les tests ──────────────
client = chromadb.Client()
# → Données perdues à la fin du programme

# ── 2. Persistant — stockage SQLite local ─────────────────
client = chromadb.PersistentClient(path="./ma_base_chroma")
# → Données sauvegardées sur disque, rechargées automatiquement

# ── 3. HTTP Client — connexion à un serveur Chroma ────────
client = chromadb.HttpClient(host="localhost", port=8001)
# → Le serveur doit tourner séparément (voir fiche 06)
```

## Pipeline complet — de zéro à la recherche

```python
import chromadb
from chromadb.utils import embedding_functions

# ── 1. Créer le client persistant ────────────────────────
client = chromadb.PersistentClient(path="./db_chroma")

# ── 2. Définir la fonction d'embedding ───────────────────
# Option A : SentenceTransformers (local, gratuit)
ef = embedding_functions.SentenceTransformerEmbeddingFunction(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)

# Option B : OpenAI (payant, très bon)
# ef = embedding_functions.OpenAIEmbeddingFunction(
#     api_key="ta-clé", model_name="text-embedding-3-small"
# )

# ── 3. Créer ou charger une collection ───────────────────
collection = client.get_or_create_collection(
    name="ma_collection",
    embedding_function=ef,
    metadata={"hnsw:space": "cosine"}  # métrique de distance : cosine (défaut)
)

# ── 4. Ajouter des documents ─────────────────────────────
collection.add(
    ids=["doc_1", "doc_2", "doc_3", "doc_4"],
    documents=[
        "La politique de retour est de 30 jours après réception.",
        "La livraison gratuite s'applique dès 50 euros d'achat.",
        "La garantie couvre 2 ans pièces et main d'œuvre.",
        "La livraison express est disponible en 24h pour 9,90€.",
    ],
    metadatas=[
        {"source": "politique.txt", "section": "retours",   "page": 1},
        {"source": "politique.txt", "section": "livraison", "page": 2},
        {"source": "politique.txt", "section": "garantie",  "page": 3},
        {"source": "politique.txt", "section": "livraison", "page": 2},
    ]
    # embeddings calculés automatiquement par ef
)

print(f"Collection : {collection.count()} documents")

# ── 5. Rechercher ─────────────────────────────────────────
résultats = collection.query(
    query_texts=["délai de retour produit"],
    n_results=2
)

print(résultats["documents"])   # → textes des chunks trouvés
print(résultats["metadatas"])   # → métadonnées associées
print(résultats["distances"])   # → distances (plus proche = plus similaire)
print(résultats["ids"])         # → IDs des documents
```

## Structure complète du résultat de query()

```python
résultats = collection.query(query_texts=["..."], n_results=3)

# résultats est un dict avec :
résultats["ids"]        # → [["doc_1", "doc_3", "doc_2"]]  liste de listes
résultats["documents"]  # → [["texte 1", "texte 3", "texte 2"]]
résultats["metadatas"]  # → [[{...}, {...}, {...}]]
résultats["distances"]  # → [[0.12, 0.34, 0.67]]  (cosine : 0=identique, 2=opposé)
résultats["embeddings"] # → None par défaut (passer include=["embeddings"] pour les avoir)

# Pour plusieurs requêtes simultanées
résultats = collection.query(
    query_texts=["retour produit", "livraison express"],
    n_results=2
)
# résultats["documents"][0] → résultats de la première requête
# résultats["documents"][1] → résultats de la deuxième requête
```

## Heartbeat — vérifier que Chroma répond

```python
client.heartbeat()   # → retourne un timestamp si le client est actif
print(client.list_collections())  # → toutes les collections existantes
```

> [!tip] get_or_create_collection()
> Toujours utiliser `get_or_create_collection()` plutôt que `create_collection()`. La première est idempotente — elle crée si absente, charge si existante. La seconde lève une erreur si la collection existe déjà.

> [!info] Embeddings automatiques
> Si tu passes une `embedding_function` à la collection, Chroma calcule automatiquement les embeddings des documents et des requêtes. Tu peux aussi passer des embeddings pré-calculés via le paramètre `embeddings=` dans `add()`.
