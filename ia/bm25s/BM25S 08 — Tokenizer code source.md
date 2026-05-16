#ia #bm25s #code #tokenizer #codebase #pratique

## Tokenizer pour code source

Le code source a des conventions spécifiques (camelCase, snake_case, identifiants) qui nécessitent un tokenizer custom. Le stemming standard dégrade les résultats sur du code.

## Pourquoi un tokenizer custom pour le code

```
Tokenizer standard sur du code :
  "getUserById"        → ["getUserById"]           ❌ 1 token opaque
  "error_handler"      → ["error_handler"]          ❌ 1 token opaque
  "DatabaseConnection" → ["DatabaseConnection"]     ❌ 1 token opaque

Tokenizer code custom :
  "getUserById"        → ["get", "user", "by", "id"]     ✅ 4 tokens
  "error_handler"      → ["error", "handler"]             ✅ 2 tokens
  "DatabaseConnection" → ["database", "connection"]       ✅ 2 tokens
```

## Tokenizer de base pour le code

```python
import re

def tokenizer_code(texte: str) -> list[str]:
    """
    Tokenizer pour code source.
    - Sépare camelCase : getUserById → get user by id
    - Sépare PascalCase : DatabaseConfig → database config
    - Sépare snake_case : error_handler → error handler
    - Sépare kebab-case : my-module → my module
    - Retire les tokens trop courts (< 3 chars)
    - Met tout en minuscules
    """
    # Séparer camelCase et PascalCase
    # "getUserById" → "get User By Id"
    texte = re.sub(r'([a-z])([A-Z])', r'\1 \2', texte)
    texte = re.sub(r'([A-Z]+)([A-Z][a-z])', r'\1 \2', texte)

    # Remplacer les séparateurs non-alpha par des espaces
    texte = re.sub(r'[_\-./\\:@#]', ' ', texte)

    # Extraire les tokens alphanumériques, tout en minuscules
    tokens = re.findall(r'[a-zA-Z][a-zA-Z0-9]*', texte)
    tokens = [t.lower() for t in tokens]

    # Filtrer les tokens trop courts (opérateurs, types courts...)
    return [t for t in tokens if len(t) >= 3]

# Tests
print(tokenizer_code("getUserById(user_id)"))
# → ['get', 'user', 'by', 'id', 'user']

print(tokenizer_code("class DatabaseConnectionPool:"))
# → ['class', 'database', 'connection', 'pool']

print(tokenizer_code("async def handle_auth_error(e: Exception) -> None:"))
# → ['async', 'def', 'handle', 'auth', 'error', 'exception', 'none']

print(tokenizer_code("raise ValueError('Invalid configuration key')"))
# → ['raise', 'value', 'error', 'invalid', 'configuration', 'key']
```

## Tokenizer avancé — code + commentaires

```python
import re

# Mots-clés de langages à exclure (peu informatifs pour la recherche)
KEYWORDS_PYTHON = {"def", "class", "import", "from", "return", "if", "else",
                   "elif", "for", "while", "try", "except", "with", "as",
                   "pass", "none", "true", "false", "and", "or", "not",
                   "in", "is", "lambda", "yield", "async", "await", "self"}

KEYWORDS_JS = {"function", "const", "let", "var", "return", "if", "else",
               "for", "while", "class", "new", "this", "async", "await",
               "import", "export", "default", "null", "undefined", "true", "false"}

def tokenizer_code_avancé(
    texte: str,
    exclure_keywords: set = KEYWORDS_PYTHON,
    min_len: int = 3
) -> list[str]:
    """
    Tokenizer code avancé avec exclusion des mots-clés du langage.
    Idéal quand on indexe des fonctions Python / JS.
    """
    # Séparer camelCase / PascalCase
    texte = re.sub(r'([a-z])([A-Z])', r'\1 \2', texte)
    texte = re.sub(r'([A-Z]+)([A-Z][a-z])', r'\1 \2', texte)

    # Remplacer les séparateurs
    texte = re.sub(r'[_\-./\\:@#()\[\]{}<>,;=!?|&*+%^~`"\']+', ' ', texte)

    # Tokeniser
    tokens = re.findall(r'[a-zA-Z][a-zA-Z0-9]*', texte)
    tokens = [t.lower() for t in tokens]

    # Filtrer
    return [
        t for t in tokens
        if len(t) >= min_len and t not in exclure_keywords
    ]

# Test
print(tokenizer_code_avancé(
    "def authenticate_user(username: str, password: str) -> Optional[User]:"
))
# → ['authenticate', 'user', 'username', 'str', 'password', 'optional', 'user']
# "def", "str" (trop court), sont exclus
```

## Pipeline BM25S pour codebase Python

```python
import bm25s

# Corpus de chunks de code — typiquement des fonctions ou classes
corpus_code = [
    # Authentification
    "def authenticate_user(username, password): verify credentials returns JWT token",
    "def refresh_token(token): validate and refresh expired authentication token",
    "class AuthMiddleware: intercept requests and validate bearer tokens",

    # Base de données
    "class DatabaseConnection: manage connection pool with retry and timeout logic",
    "def execute_query(query, params): run parameterized SQL with error handling",
    "def migrate_schema(version): apply database migrations in transaction",

    # API
    "def handle_request(endpoint, method): route HTTP requests to controllers",
    "class APIRateLimiter: enforce request rate limits per user and IP",
    "def serialize_response(data): convert models to JSON API format",

    # Tests
    "def run_test_suite(module): execute pytest with fixtures and coverage",
    "class MockDatabase: in-memory database for unit testing",
    "def assert_response_schema(response, schema): validate API response structure",
]

# Indexer SANS stemmer, AVEC tokenizer code
corpus_tokenisé = bm25s.tokenize(
    corpus_code,
    tokenizer=tokenizer_code_avancé
    # Pas de stopwords ni de stemmer pour le code
)

retriever_code = bm25s.BM25()
retriever_code.index(corpus_tokenisé)
retriever_code.save("./index_code", corpus=corpus_code)

# Test
for query in ["authentication token validation", "database connection retry", "API rate limit"]:
    q = bm25s.tokenize(query, tokenizer=tokenizer_code_avancé)
    résultats, scores = retriever_code.retrieve(q, corpus=corpus_code, k=2)
    print(f"\n🔍 '{query}'")
    for i in range(résultats.shape[1]):
        print(f"  {scores[0,i]:.2f} | {résultats[0,i][:80]}")
```

## Retriever LangChain — code source

```python
from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain.text_splitter import Language, RecursiveCharacterTextSplitter

# TextSplitter spécialisé pour le code Python
# Coupe aux bons endroits : classes, fonctions, méthodes
splitter_python = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,
    chunk_size=1000,    # plus grand pour le code — garder les fonctions entières
    chunk_overlap=100
)

# Charger tous les fichiers Python
loader = DirectoryLoader("src/", glob="**/*.py", loader_cls=TextLoader)
docs_code = loader.load()
chunks_code = splitter_python.split_documents(docs_code)

# BM25SRetriever sans stemmer, avec tokenizer code
retriever_code = BM25SRetriever.from_documents(
    chunks_code,
    avec_stemming=False,                    # ← pas de stemming
    tokenizer_fn=tokenizer_code_avancé,     # ← tokenizer code
    k=4,
    index_path="./index_code"
)
```

## Hybrid complet — docs EN + codebase

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.vectorstores import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings
import Stemmer

stemmer_en = Stemmer.Stemmer("english")

# ── Retriever 1 : BM25S pour les docs (avec stemming) ────
retriever_bm25_docs = BM25SRetriever.from_documents(
    chunks_docs, langue="english", avec_stemming=True, k=3,
    index_path="./index_docs"
)

# ── Retriever 2 : BM25S pour le code (sans stemming) ─────
retriever_bm25_code = BM25SRetriever.from_documents(
    chunks_code, avec_stemming=False, tokenizer_fn=tokenizer_code_avancé, k=3,
    index_path="./index_code"
)

# ── Retriever 3 : Vectoriel (sémantique) sur tout ─────────
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"  # anglais
)
vectorstore = Chroma.from_documents(
    chunks_docs + chunks_code, embeddings, persist_directory="./db"
)
retriever_vectoriel = vectorstore.as_retriever(search_kwargs={"k": 4})

# ── Fusion finale ─────────────────────────────────────────
hybrid_final = EnsembleRetriever(
    retrievers=[retriever_bm25_docs, retriever_bm25_code, retriever_vectoriel],
    weights=[0.25, 0.25, 0.50]   # vectoriel dominant pour la sémantique
)

# RAG Chain sur toute la base de connaissance
chain = (
    {"contexte": hybrid_final | formater_docs, "question": RunnablePassthrough()}
    | prompt | llm | StrOutputParser()
)

# Fonctionne sur les deux types de questions
chain.invoke("How does authentication work?")          # → docs
chain.invoke("What does authenticate_user() return?")  # → code
chain.invoke("How to handle database connection errors?")  # → les deux
```

## Choisir l'embedding model pour le code

```python
# Pour une codebase anglaise, les embeddings spécialisés code sont meilleurs
# Ils comprennent les identifiants, patterns de code, semantique technique

# Option 1 — Spécialisé code (recommandé)
from langchain_community.embeddings import HuggingFaceEmbeddings
embeddings_code = HuggingFaceEmbeddings(
    model_name="microsoft/codebert-base"        # spécialisé code
)

# Option 2 — Généraliste anglais (bon compromis docs + code)
embeddings_general = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"   # rapide, anglais
)

# Option 3 — Multilingue (si tu as du code avec commentaires en français)
embeddings_multi = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)
```

## Récapitulatif — quelle config pour quoi

```
Contenu                Config BM25S                    Embedding
────────────────────────────────────────────────────────────────────────
Docs anglaises    →    stemmer=english, stopwords=en    all-MiniLM-L6-v2
Code Python/JS    →    pas de stemmer, tokenizer code   codebert-base
Docs + Code EN    →    2 retrievers BM25S séparés       all-MiniLM-L6-v2
Docs FR           →    stemmer=french, stopwords=fr     multilingual-MiniLM
Docs FR + EN      →    2 retrievers BM25S séparés       multilingual-MiniLM
```

> [!tip] RecursiveCharacterTextSplitter.from_language()
> Pour les codebases, utilise ce splitter spécialisé plutôt que le splitter générique. Il coupe aux frontières naturelles du code (fin de fonction, fin de classe) et garde les blocs sémantiquement cohérents.

> [!warning] Pas de stemming sur les identifiants
> Le stemmer anglais peut transformer "config" → "configur", "error" → "error", mais aussi "database" → "databas". Ces transformations détruisent la recherche exacte sur les noms de classes et fonctions. Toujours désactiver le stemmer sur du code pur.
