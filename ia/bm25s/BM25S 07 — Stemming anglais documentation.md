#ia #bm25s #anglais #stemming #documentation #pratique

## Stemming anglais — documentation technique

Le stemming anglais avec PyStemmer (Snowball) fonctionne exactement comme le français — seul le nom de la langue change.

## Langues supportées par PyStemmer

```python
import Stemmer

# Voir toutes les langues disponibles
print(Stemmer.algorithms())
# → ['arabic', 'danish', 'dutch', 'english', 'finnish', 'french',
#    'german', 'hungarian', 'italian', 'norwegian', 'porter',
#    'portuguese', 'romanian', 'russian', 'spanish', 'swedish',
#    'tamil', 'turkish']

# Créer les stemmers
stemmer_en = Stemmer.Stemmer("english")
stemmer_fr = Stemmer.Stemmer("french")
stemmer_de = Stemmer.Stemmer("german")
```

## Ce que fait le stemmer anglais

```python
import Stemmer

stemmer_en = Stemmer.Stemmer("english")

# Variantes d'un même concept → même racine
print(stemmer_en.stemWords([
    "running", "runs", "runner",           # → run, run, runner
    "documentation", "documented", "docs", # → document, document, doc
    "configuration", "configuring", "configured", # → configur, configur, configur
    "authentication", "authenticating",    # → authent, authent
    "deployment", "deploying", "deployed", # → deploy, deploy, deploy
    "testing", "tested", "tests",          # → test, test, test
]))
# → ['run', 'run', 'runner', 'document', 'document', 'doc',
#    'configur', 'configur', 'configur', 'authent', 'authent',
#    'deploy', 'deploy', 'deploy', 'test', 'test', 'test']
```

## Pipeline BM25S pour documentation anglaise

```python
import bm25s
import Stemmer

stemmer_en = Stemmer.Stemmer("english")

corpus_docs = [
    "How to configure the authentication module and manage sessions.",
    "Authentication tokens expire after 24 hours by default.",
    "Configuring database connection pooling and retry policies.",
    "Database schema migration guide for production deployments.",
    "Running unit tests with pytest and generating coverage reports.",
    "Test suite configuration and continuous integration setup.",
    "Deployment guide for Kubernetes clusters and Docker containers.",
    "Error handling best practices and logging configuration.",
]

# Tokeniser avec stemming anglais
corpus_tokenisé = bm25s.tokenize(
    corpus_docs,
    stemmer=stemmer_en,
    stopwords="english"   # stopwords anglais intégrés dans bm25s
)

retriever = bm25s.BM25()
retriever.index(corpus_tokenisé)
retriever.save("./index_docs_en", corpus=corpus_docs)

def rechercher(query: str, k: int = 3) -> list:
    q = bm25s.tokenize(query, stemmer=stemmer_en, stopwords="english")
    résultats, scores = retriever.retrieve(q, corpus=corpus_docs, k=k)
    return [(résultats[0, i], float(scores[0, i])) for i in range(résultats.shape[1])]

# Tests — le stemming fusionne les variantes
for query in ["configuring authentication", "deploy application", "test coverage"]:
    print(f"\n🔍 '{query}'")
    for texte, score in rechercher(query):
        print(f"  {score:.2f} | {texte[:70]}")

# → "configuring" trouve "configure", "configuration", "configuring"
# → "deploy" trouve "deployment", "deploying", "deployed"
# → "test" trouve "testing", "tested", "tests"
```

## Retriever LangChain — docs anglaises

```python
from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Charger la documentation
loader = DirectoryLoader(
    "docs/",
    glob="**/*.md",
    loader_cls=TextLoader
)
docs = loader.load()

splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,
    chunk_overlap=50,
    separators=["\n\n", "\n", ". ", " ", ""]
)
chunks = splitter.split_documents(docs)

# BM25SRetriever en anglais
retriever_docs = BM25SRetriever.from_documents(
    chunks,
    k=4,
    langue="english",       # ← anglais
    avec_stemming=True,
    index_path="./index_docs_en"
)

# Tester
résultats = retriever_docs.invoke("how to configure authentication")
for doc in résultats:
    print(f"Score: {doc.metadata['bm25s_score']:.3f} | {doc.page_content[:100]}")
```

## Corpus multilingue FR + EN — deux stratégies

### Stratégie A — Deux retrievers séparés fusionnés (recommandée)

```python
from langchain.retrievers import EnsembleRetriever

# Retriever anglais — chunks de docs EN
retriever_en = BM25SRetriever.from_documents(
    chunks_en,
    langue="english",
    avec_stemming=True,
    k=3,
    index_path="./index_en"
)

# Retriever français — chunks de docs FR
retriever_fr = BM25SRetriever.from_documents(
    chunks_fr,
    langue="french",
    avec_stemming=True,
    k=3,
    index_path="./index_fr"
)

# Fusion — la requête est envoyée aux deux
hybrid_bilingue = EnsembleRetriever(
    retrievers=[retriever_en, retriever_fr],
    weights=[0.5, 0.5]   # ajuster selon la proportion EN/FR du corpus
)

# Fonctionne quelle que soit la langue de la requête
résultats = hybrid_bilingue.invoke("authentication configuration")
résultats = hybrid_bilingue.invoke("configuration de l'authentification")
```

### Stratégie B — Corpus mixte sans stemmer (compromis)

```python
# Sans stemmer → perte de recall mais cohérence garantie
retriever_mixte = BM25SRetriever.from_documents(
    chunks_en + chunks_fr,
    avec_stemming=False,   # ← pas de stemmer sur corpus bilingue
    index_path="./index_mixte"
)
# Moins bon recall mais évite les conflits entre stemmers
```

> [!tip] Stratégie A > Stratégie B
> Deux retrievers séparés avec leurs stemmers respectifs donnent de bien meilleurs résultats qu'un corpus mixte sans stemmer. Le coût est minimal (deux petits index sur disque).

> [!info] "porter" vs "english"
> PyStemmer propose deux stemmers anglais : `"porter"` (ancien, algorithme Porter) et `"english"` (Snowball, plus moderne et précis). Utilise toujours `"english"` sauf compatibilité avec un système existant.
