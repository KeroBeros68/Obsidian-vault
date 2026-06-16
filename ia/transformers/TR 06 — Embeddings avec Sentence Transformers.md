#ia #transformers #embeddings #sentence-transformers #rag #pratique

## Embeddings avec Sentence Transformers

`sentence-transformers` est la librairie de référence pour générer des embeddings sémantiques de haute qualité. C'est ce qui tourne derrière `HuggingFaceEmbeddings` de LangChain.

## Installation

```bash
pip install sentence-transformers
```

## Usage basique

```python
from sentence_transformers import SentenceTransformer

# Charger un modèle
model = SentenceTransformer("sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2")

# Encoder des textes → vecteurs
phrases = [
    "La politique de retour est de 30 jours.",
    "Comment retourner un article défectueux ?",
    "Recette du gâteau au chocolat.",
]

embeddings = model.encode(phrases)
print(embeddings.shape)   # → (3, 384)  ← 3 phrases × 384 dimensions
```

## Calculer la similarité

```python
from sentence_transformers import SentenceTransformer, util
import torch

model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")

# Phrases à comparer
phrase1 = "Comment retourner un produit ?"
phrases = [
    "La politique de retour est de 30 jours.",   # très similaire
    "La livraison express coûte 9,90€.",          # peu similaire
    "Les retours sont acceptés sous 30 jours.",   # très similaire
    "Recette de tarte aux pommes.",               # pas du tout similaire
]

# Encoder
emb1   = model.encode(phrase1,  convert_to_tensor=True)
emb2   = model.encode(phrases,  convert_to_tensor=True)

# Similarité cosinus
scores = util.cos_sim(emb1, emb2)
print(scores)
# → tensor([[0.85, 0.12, 0.82, 0.03]])

# Trouver les plus similaires
top_results = torch.topk(scores[0], k=2)
for idx, score in zip(top_results.indices, top_results.values):
    print(f"{score:.3f} | {phrases[idx]}")
# → 0.850 | La politique de retour est de 30 jours.
# → 0.820 | Les retours sont acceptés sous 30 jours.
```

## Choisir le bon modèle d'embedding

```python
# ── Modèles multilingues (FR + EN + autres) ───────────────
"sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
# → 50 langues, 384 dims, rapide, bon en français

"sentence-transformers/paraphrase-multilingual-mpnet-base-v2"
# → 50 langues, 768 dims, meilleur mais plus lent

"intfloat/multilingual-e5-large"
# → Très performant, 1024 dims, nécessite un préfixe "query:" / "passage:"

# ── Modèles anglais uniquement ────────────────────────────
"sentence-transformers/all-MiniLM-L6-v2"
# → Anglais, 384 dims, ultra-rapide, bon équilibre

"sentence-transformers/all-mpnet-base-v2"
# → Anglais, 768 dims, meilleur en qualité

"BAAI/bge-large-en-v1.5"
# → Anglais, 1024 dims, top du classement MTEB

# ── Modèles spécialisés code ──────────────────────────────
"microsoft/codebert-base"
# → Code source, 768 dims

"flax-sentence-embeddings/st-codesearch-distilroberta-base"
# → Recherche de code, 768 dims
```

## Encoder avec préfixes (modèles E5, BGE)

Certains modèles performants nécessitent un préfixe pour distinguer query et document.

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("intfloat/multilingual-e5-large")

# Les requêtes doivent être préfixées avec "query: "
# Les documents avec "passage: "
query      = "query: Comment retourner un article ?"
documents  = [
    "passage: La politique de retour est de 30 jours.",
    "passage: La livraison express coûte 9,90€.",
]

emb_query = model.encode([query])
emb_docs  = model.encode(documents)

scores = util.cos_sim(emb_query, emb_docs)
print(scores)   # → [[0.89, 0.21]]
```

## Batch encoding et performance

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")

# Encoder un grand corpus efficacement
corpus_large = ["texte " + str(i) for i in range(10000)]

embeddings = model.encode(
    corpus_large,
    batch_size=64,          # traiter 64 phrases à la fois
    show_progress_bar=True, # barre de progression
    convert_to_numpy=True,  # numpy array (défaut)
    normalize_embeddings=True  # normaliser → similarité cosinus = produit scalaire
)
print(embeddings.shape)   # → (10000, 384)
```

## Intégration directe avec Chroma

```python
from sentence_transformers import SentenceTransformer
import chromadb
from chromadb.utils.embedding_functions import EmbeddingFunction

# Wrapper custom pour Chroma
class STEmbeddingFunction(EmbeddingFunction):
    def __init__(self, model_name: str):
        self.model = SentenceTransformer(model_name)

    def __call__(self, input: list[str]) -> list[list[float]]:
        return self.model.encode(input, normalize_embeddings=True).tolist()

# Utiliser avec Chroma
ef = STEmbeddingFunction("paraphrase-multilingual-MiniLM-L12-v2")
client     = chromadb.PersistentClient("./db")
collection = client.get_or_create_collection("docs", embedding_function=ef)

collection.add(
    ids=["1", "2"],
    documents=["Politique de retour...", "Livraison gratuite..."]
)
```

## Intégration LangChain

```python
from langchain_community.embeddings import HuggingFaceEmbeddings

# HuggingFaceEmbeddings utilise sentence-transformers sous le capot
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2",
    model_kwargs={"device": "cuda"},  # GPU
    encode_kwargs={"normalize_embeddings": True, "batch_size": 64}
)

# Test direct
vecteur = embeddings.embed_query("Comment retourner un produit ?")
print(len(vecteur))   # → 384

vecteurs = embeddings.embed_documents(["texte 1", "texte 2"])
print(len(vecteurs), len(vecteurs[0]))   # → 2, 384
```

## Évaluer la qualité d'un modèle d'embedding

```python
from sentence_transformers.evaluation import InformationRetrievalEvaluator

# Dataset d'évaluation
queries   = {"q1": "retour produit",       "q2": "livraison gratuite"}
corpus    = {"d1": "30 jours pour retourner...", "d2": "gratuit dès 50€..."}
relevant  = {"q1": {"d1"}, "q2": {"d2"}}   # quels docs sont pertinents par query

evaluator = InformationRetrievalEvaluator(
    queries=queries,
    corpus=corpus,
    relevant_docs=relevant
)

score = evaluator(model)
print(f"NDCG@10 : {score:.4f}")   # → score entre 0 et 1
```

> [!tip] normalize_embeddings=True
> Avec les embeddings normalisés (norme = 1), la similarité cosinus devient équivalente au produit scalaire — beaucoup plus rapide à calculer. Toujours activer sauf raison contraire.

> [!info] Classement MTEB
> Le MTEB (Massive Text Embedding Benchmark) est le classement de référence pour comparer les modèles d'embedding. Consulte huggingface.co/spaces/mteb/leaderboard pour choisir le meilleur modèle selon ta langue et ta tâche.
