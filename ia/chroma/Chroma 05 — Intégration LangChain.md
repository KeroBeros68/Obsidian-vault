#ia #chroma #langchain #intégration #rag #pratique

## Intégration LangChain

Chroma s'utilise sans credentials dans LangChain — installer le package suffit. L'intégration est fournie par `langchain-chroma`, un package séparé depuis LangChain v0.2.

## Installation

```bash
pip install langchain-chroma chromadb
```

## Créer un vectorstore Chroma via LangChain

```python
from langchain_chroma import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)

# ── Depuis des Documents LangChain ────────────────────────
from langchain_community.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

loader   = TextLoader("politique.txt", encoding="utf-8")
docs     = loader.load()
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=50)
chunks   = splitter.split_documents(docs)

# Crée la collection + indexe les chunks en une ligne
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    collection_name="politique_retour",
    persist_directory="./db_langchain"   # persistance automatique
)

# ── Recharger depuis le disque (session suivante) ─────────
vectorstore = Chroma(
    collection_name="politique_retour",
    embedding_function=embeddings,
    persist_directory="./db_langchain"
)
```

## Depuis un client Chroma existant

```python
import chromadb
from langchain_chroma import Chroma

# Initialiser depuis un client Chroma custom
client = chromadb.PersistentClient(path="./db")
collection = client.get_or_create_collection("ma_collection")

# Passer le client existant à LangChain
vectorstore = Chroma(
    client=client,
    collection_name="ma_collection",
    embedding_function=embeddings
)
```

## CRUD via LangChain

```python
from langchain_core.documents import Document

# ── Ajouter des documents ─────────────────────────────────
doc1 = Document(page_content="Nouveau texte 1", metadata={"source": "nouveau.pdf"})
doc2 = Document(page_content="Nouveau texte 2", metadata={"source": "nouveau.pdf"})

ids = vectorstore.add_documents([doc1, doc2])
print(ids)   # → IDs générés automatiquement

# Ajouter avec IDs custom
vectorstore.add_documents(
    [doc1, doc2],
    ids=["mon_id_1", "mon_id_2"]
)

# ── Mettre à jour ─────────────────────────────────────────
doc_mis_à_jour = Document(
    page_content="Texte mis à jour",
    metadata={"source": "nouveau.pdf", "version": 2}
)
vectorstore.update_documents(ids=["mon_id_1"], documents=[doc_mis_à_jour])

# ── Supprimer ─────────────────────────────────────────────
vectorstore.delete(ids=["mon_id_1", "mon_id_2"])
```

## Recherche via LangChain

```python
# Recherche par similarité
résultats = vectorstore.similarity_search(
    "retour produit défectueux",
    k=3
)
for doc in résultats:
    print(doc.page_content[:100])
    print(doc.metadata)

# Avec scores
résultats_scored = vectorstore.similarity_search_with_score(
    "retour produit",
    k=3
)
for doc, score in résultats_scored:
    print(f"Score: {score:.3f} | {doc.page_content[:80]}")

# Avec filtre metadata
résultats_filtrés = vectorstore.similarity_search(
    "configuration",
    k=3,
    filter={"source": "guide.md"}   # filtre Chroma natif
)

# MMR — Maximum Marginal Relevance (résultats diversifiés)
résultats_mmr = vectorstore.max_marginal_relevance_search(
    "retour produit",
    k=4,
    fetch_k=20,      # candidats à considérer
    lambda_mult=0.7  # 0=max diversité, 1=max similarité
)
```

## Retriever LangChain depuis Chroma

```python
# Retriever basique
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}
)

# Retriever avec filtre
retriever_filtré = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={
        "k": 4,
        "filter": {"langue": "fr"}
    }
)

# Retriever MMR
retriever_mmr = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={"k": 4, "fetch_k": 20}
)

# Retriever avec seuil de score
retriever_seuil = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"score_threshold": 0.7, "k": 10}
)
```

## RAG Chain complète avec Chroma

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-sonnet-4-20250514")

prompt = ChatPromptTemplate.from_messages([
    ("system", """Tu es un assistant expert.
Réponds uniquement depuis le contexte fourni.
Cite toujours la source entre crochets.

Contexte :
{contexte}"""),
    ("human", "{question}")
])

def formater_docs(docs):
    return "\n\n---\n\n".join([
        f"[{doc.metadata.get('source','?')} p.{doc.metadata.get('page','?')}]\n{doc.page_content}"
        for doc in docs
    ])

retriever = vectorstore.as_retriever(search_kwargs={"k": 4})

chain = (
    {"contexte": retriever | formater_docs, "question": RunnablePassthrough()}
    | prompt | llm | StrOutputParser()
)

réponse = chain.invoke("Quelle est la politique de retour ?")
print(réponse)
```

## Chroma Cloud via LangChain

```python
import os
from langchain_chroma import Chroma

# Connexion à Chroma Cloud
vectorstore = Chroma(
    collection_name="ma_collection",
    embedding_function=embeddings,
    chroma_cloud_api_key=os.getenv("CHROMA_API_KEY"),
    tenant=os.getenv("CHROMA_TENANT"),
    database=os.getenv("CHROMA_DATABASE"),
)
# Même API qu'en local — aucun changement de code
```

> [!tip] from_documents() vs Chroma()
> `Chroma.from_documents()` = crée un vectorstore depuis scratch. `Chroma()` = se connecte à un vectorstore existant. En production, utilise `from_documents()` une fois à l'indexation, puis `Chroma()` à chaque redémarrage.

> [!warning] Nouveau package langchain-chroma
> Depuis LangChain v0.2, importe depuis `langchain_chroma` (pas `langchain_community.vectorstores`). L'ancien import fonctionne encore mais est déprécié. Migre vers `from langchain_chroma import Chroma`.
