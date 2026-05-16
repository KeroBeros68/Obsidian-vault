#ia #bm25s #persistance #huggingface #pratique

## Persistance et Hugging Face Hub

L'un des avantages majeurs de bm25s sur rank-bm25 : l'index peut être sauvegardé sur disque et rechargé sans recalcul.

## Sauvegarder l'index sur disque

```python
import bm25s
import Stemmer

corpus = [
    "La politique de retour est de 30 jours.",
    "Livraison gratuite dès 50 euros.",
    "Garantie 2 ans pièces et main d'œuvre.",
]

stemmer = Stemmer.Stemmer("french")
corpus_tokenisé = bm25s.tokenize(corpus, stemmer=stemmer, stopwords="french")

retriever = bm25s.BM25()
retriever.index(corpus_tokenisé)

# Sauvegarder l'index + le corpus
retriever.save(
    "./mon_index_bm25s",   # dossier de destination (créé automatiquement)
    corpus=corpus          # optionnel : sauvegarde aussi les textes originaux
)

# Structure créée :
# mon_index_bm25s/
#   ├── data.index.npy       ← matrice sparse des scores
#   ├── params.index.json    ← paramètres BM25 (k1, b, method...)
#   └── corpus.jsonl         ← textes originaux (si corpus fourni)
```

## Recharger l'index

```python
import bm25s

# Recharger sans les textes (retourne des indices)
retriever = bm25s.BM25.load("./mon_index_bm25s")

# Recharger AVEC les textes (retourne directement les textes)
retriever = bm25s.BM25.load(
    "./mon_index_bm25s",
    load_corpus=True   # ← charge corpus.jsonl automatiquement
)

# Utilisation identique après rechargement
query = bm25s.tokenize("retour produit")
résultats, scores = retriever.retrieve(query, k=3)
# Si load_corpus=True → résultats contient directement les textes
```

## Mémoire mappée (memory-mapped) — grands corpus

Pour les index très volumineux, éviter de tout charger en RAM.

```python
# Charger en mémoire mappée — les données restent sur disque
# et sont lues à la demande (économise la RAM)
retriever = bm25s.BM25.load(
    "./mon_index_bm25s",
    load_corpus=True,
    mmap=True   # ← lecture sur disque à la demande
)
# Idéal pour les corpus de plusieurs GB
```

## Partager sur Hugging Face Hub

```python
import bm25s
from huggingface_hub import HfApi

# 1. Construire et sauvegarder l'index localement
retriever.save("./mon_index", corpus=corpus)

# 2. Pousser sur le Hub
api = HfApi()
api.upload_folder(
    folder_path="./mon_index",
    repo_id="mon-organisation/mon-index-bm25s",
    repo_type="model"
)
# → Disponible sur huggingface.co/mon-organisation/mon-index-bm25s
```

## Charger depuis Hugging Face Hub

```python
import bm25s

# Charger directement depuis le Hub (téléchargement automatique)
retriever = bm25s.BM25.load(
    "mon-organisation/mon-index-bm25s",
    load_corpus=True
)

# Utiliser immédiatement
query = bm25s.tokenize("politique de retour")
résultats, scores = retriever.retrieve(query, k=3)
```

## Workflow production complet

```python
import bm25s
import Stemmer
import os

INDEX_PATH = "./index_production"

def construire_ou_charger_index(corpus: list[str]) -> tuple:
    """Charge l'index existant ou le reconstruit si absent."""
    stemmer = Stemmer.Stemmer("french")

    if os.path.exists(INDEX_PATH):
        print("Chargement de l'index existant...")
        retriever = bm25s.BM25.load(INDEX_PATH, load_corpus=True)
    else:
        print("Construction de l'index...")
        corpus_tokenisé = bm25s.tokenize(
            corpus, stemmer=stemmer, stopwords="french"
        )
        retriever = bm25s.BM25()
        retriever.index(corpus_tokenisé)
        retriever.save(INDEX_PATH, corpus=corpus)
        print(f"Index sauvegardé dans {INDEX_PATH}")

    return retriever, stemmer

def mettre_à_jour_index(nouveaux_docs: list[str]):
    """Reconstruit l'index avec de nouveaux documents."""
    # BM25S ne supporte pas l'ajout incrémental — reconstruire entièrement
    import shutil
    if os.path.exists(INDEX_PATH):
        shutil.rmtree(INDEX_PATH)
    construire_ou_charger_index(nouveaux_docs)

# Utilisation
corpus = ["doc 1...", "doc 2...", "doc 3..."]
retriever, stemmer = construire_ou_charger_index(corpus)
```

> [!warning] Pas d'ajout incrémental
> BM25S ne supporte pas l'ajout de documents à un index existant. Pour mettre à jour le corpus, il faut reconstruire l'index entièrement. Planifie des reconstructions périodiques (nuit, weekend) si ton corpus évolue.

> [!tip] Mmap pour les grands index
> Si ton index fait plusieurs GB, utilise `mmap=True` au chargement. L'index reste sur disque et seules les parties nécessaires à la requête sont lues en RAM. Idéal pour les serveurs avec peu de RAM.
