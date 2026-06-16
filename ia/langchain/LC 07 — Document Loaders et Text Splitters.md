#ia #langchain #rag #loaders #splitters #bases

## Document Loaders et Text Splitters

## Document Loaders — charger les sources

```python
from langchain_community.document_loaders import (
    TextLoader,         # fichiers .txt
    PyPDFLoader,        # fichiers PDF
    DirectoryLoader,    # dossier entier
    WebBaseLoader,      # pages web
    CSVLoader,          # fichiers CSV
    JSONLoader,         # fichiers JSON
    UnstructuredFileLoader  # tout type de fichier
)

# Fichier texte
loader = TextLoader("doc.txt", encoding="utf-8")
docs = loader.load()
# → [Document(page_content="...", metadata={"source": "doc.txt"})]

# PDF — 1 Document par page
loader = PyPDFLoader("rapport.pdf")
docs = loader.load()
# → metadata contient "source" et "page"

# Dossier entier
loader = DirectoryLoader("mes_docs/", glob="**/*.txt", loader_cls=TextLoader)
docs = loader.load()

# Page web
loader = WebBaseLoader("https://python.org/about")
docs = loader.load()

# CSV — 1 Document par ligne
loader = CSVLoader("données.csv", source_column="url")
docs = loader.load()
```

> [!info] Un Document LangChain
> `page_content` (le texte) + `metadata` (source, page, date...). Les métadonnées permettent de citer les sources dans les réponses RAG.

## Text Splitters — découper en chunks

```python
# pip install langchain-text-splitters
from langchain_text_splitters import (
    RecursiveCharacterTextSplitter,   # le plus utilisé
    CharacterTextSplitter,            # coupe sur un seul séparateur
    TokenTextSplitter,                # coupe par nombre de tokens
    MarkdownHeaderTextSplitter,       # respecte la structure Markdown
    HTMLHeaderTextSplitter            # respecte les balises HTML
)

# RecursiveCharacterTextSplitter — recommandé par défaut
splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,        # taille max en caractères
    chunk_overlap=50,      # chevauchement entre chunks
    separators=["\n\n", "\n", ". ", " ", ""]  # ordre de priorité de coupe
)

chunks = splitter.split_documents(docs)
print(f"{len(docs)} docs → {len(chunks)} chunks")
print(chunks[0].page_content)
print(chunks[0].metadata)   # → source préservée !

# MarkdownHeaderTextSplitter — respecte la hiérarchie des titres
from langchain_text_splitters import MarkdownHeaderTextSplitter

splitter_md = MarkdownHeaderTextSplitter(
    headers_to_split_on=[
        ("#", "titre"),
        ("##", "section"),
        ("###", "sous_section")
    ]
)
chunks_md = splitter_md.split_text(texte_markdown)
# → metadata contient "titre", "section", "sous_section"
```

> [!warning] Package séparé depuis LangChain 0.1
> `langchain.text_splitter` a été extrait dans un package dédié. `pip install langchain-text-splitters` est requis, puis `from langchain_text_splitters import ...`.

> [!tip] Paramètres par défaut
> `chunk_size=512` et `chunk_overlap=50` sont de bons points de départ. Augmente si les réponses manquent de contexte, diminue si elles contiennent trop de bruit.
