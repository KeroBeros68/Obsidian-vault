#ia #bm25s #langchain #rag #hybrid #production #pratique

## Hybrid RAG BM25S + vectoriel

Combine BM25S (mots-clés exacts, rapide) et un vectorstore (sens sémantique) via EnsembleRetriever pour une couverture maximale.

## Pourquoi c'est l'approche recommandée en production

```
BM25S seul :
  ✅ "code erreur ERR_502" → trouve exactement "ERR_502"
  ❌ "panne de démarrage" → ne trouve pas "voiture ne démarre pas"

Vectoriel seul :
  ✅ "panne de démarrage" → trouve "voiture ne démarre pas" (même sens)
  ❌ "ERR_502" → peut trouver "problèmes de connexion" plutôt que le code exact

BM25S + Vectoriel (Hybrid) :
  ✅ Les deux cas → meilleure couverture que l'un ou l'autre seul
```

## Pipeline Hybrid RAG complet

```python
import bm25s
import Stemmer
from langchain_core.retrievers import BaseRetriever
from langchain_core.documents import Document
from langchain_core.callbacks import CallbackManagerForRetrieverRun
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain.retrievers import EnsembleRetriever
from langchain_community.vectorstores import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_anthropic import ChatAnthropic
from pydantic import Field
from typing import List, Optional

# ── 1. Charger et découper ────────────────────────────────
loader  = TextLoader("politique.txt", encoding="utf-8")
docs    = loader.load()
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=50)
chunks  = splitter.split_documents(docs)

# ── 2. BM25S Retriever ────────────────────────────────────
# (utiliser la classe BM25SRetriever de la fiche LC 05)
bm25s_retriever = BM25SRetriever.from_documents(
    chunks,
    k=4,
    langue="french",
    avec_stemming=True,
    index_path="./index_bm25s"
)

# ── 3. Vectorstore + Retriever ────────────────────────────
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)
vectorstore = Chroma.from_documents(
    chunks, embeddings, persist_directory="./db_chroma"
)
vectoriel_retriever = vectorstore.as_retriever(search_kwargs={"k": 4})

# ── 4. Fusion RRF ─────────────────────────────────────────
hybrid_retriever = EnsembleRetriever(
    retrievers=[bm25s_retriever, vectoriel_retriever],
    weights=[0.4, 0.6]   # BM25S=40%, vectoriel=60%
)

# ── 5. RAG Chain ──────────────────────────────────────────
llm = ChatAnthropic(model="claude-sonnet-4-20250514")

prompt = ChatPromptTemplate.from_messages([
    ("system", """Tu es un assistant expert et factuel.
Règles :
- Réponds UNIQUEMENT depuis le contexte fourni
- Si l'information est absente, dis-le clairement
- Cite toujours la source entre crochets [source]

Contexte :
{contexte}"""),
    ("human", "{question}")
])

def formater_docs(docs):
    vus = set()
    résultat = []
    for doc in docs:
        # Dédupliquer si le même chunk est retourné par les deux retrievers
        clé = doc.page_content[:100]
        if clé not in vus:
            vus.add(clé)
            source = doc.metadata.get("source", "?")
            page   = doc.metadata.get("page", "?")
            score  = doc.metadata.get("bm25s_score", "")
            score_str = f" | BM25={score:.2f}" if score else ""
            résultat.append(f"[{source} p.{page}{score_str}]\n{doc.page_content}")
    return "\n\n---\n\n".join(résultat)

chain_hybrid = (
    {"contexte": hybrid_retriever | formater_docs, "question": RunnablePassthrough()}
    | prompt | llm | StrOutputParser()
)

# ── 6. Test ───────────────────────────────────────────────
questions = [
    "Quel est le délai de retour ?",
    "La livraison express coûte combien ?",
    "Que couvre la garantie ?",
    "Quel est le seuil pour la livraison gratuite ?"
]

for q in questions:
    print(f"\n❓ {q}")
    print(f"💬 {chain_hybrid.invoke(q)}")
```

## Tuner les poids selon le type de contenu

```python
# Documentation technique — codes, noms propres, identifiants
hybrid_tech = EnsembleRetriever(
    retrievers=[bm25s_retriever, vectoriel_retriever],
    weights=[0.6, 0.4]   # BM25S dominant
)

# FAQ / support client — questions vagues, synonymes
hybrid_support = EnsembleRetriever(
    retrievers=[bm25s_retriever, vectoriel_retriever],
    weights=[0.3, 0.7]   # vectoriel dominant
)

# Contrats juridiques — termes exacts + contexte
hybrid_legal = EnsembleRetriever(
    retrievers=[bm25s_retriever, vectoriel_retriever],
    weights=[0.5, 0.5]   # équilibre
)
```

## Avec mémoire — chatbot documentaire hybride

```python
from langchain_core.prompts import MessagesPlaceholder
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
from operator import itemgetter

prompt_mémo = ChatPromptTemplate.from_messages([
    ("system", """Tu es un assistant documentaire.
Réponds depuis le contexte. Tiens compte de l'historique.

Contexte : {contexte}"""),
    MessagesPlaceholder("history"),
    ("human", "{question}")
])

chain_mémo = (
    {
        "contexte": itemgetter("question") | hybrid_retriever | formater_docs,
        "question": itemgetter("question"),
        "history":  itemgetter("history")
    }
    | prompt_mémo | llm | StrOutputParser()
)

sessions = {}
chain_finale = RunnableWithMessageHistory(
    chain_mémo,
    lambda sid: sessions.setdefault(sid, InMemoryChatMessageHistory()),
    input_messages_key="question",
    history_messages_key="history"
)

config = {"configurable": {"session_id": "user1"}}
chain_finale.invoke({"question": "Quel est le délai de retour ?"}, config=config)
chain_finale.invoke({"question": "Et pour le remboursement ?"}, config=config)
chain_finale.invoke({"question": "Résume ce qu'on a vu"}, config=config)
```

## Récapitulatif — architecture complète

```
Documents (txt, PDF, CSV...)
        ↓
TextSplitter (512 tokens, overlap 50)
        ↓
        ├── BM25SRetriever (stemming fr, k=4, index persisté)
        └── Chroma + HuggingFaceEmbeddings (vectoriel, k=4)
                ↓
        EnsembleRetriever RRF [0.4, 0.6]
                ↓
        formater_docs() + déduplication
                ↓
        ChatPromptTemplate (system + contexte + question)
                ↓
        LLM (Claude, GPT, vLLM...)
                ↓
        StrOutputParser()
                ↓
        Réponse sourcée et vérifiable
```

> [!tip] Toujours dédupliquer les chunks
> EnsembleRetriever peut retourner le même chunk depuis BM25S ET depuis le vectorstore. La fonction `formater_docs()` avec déduplication évite que le LLM voie le même texte deux fois.

> [!tip] BM25S + MMR pour encore plus de diversité
> Combine le retriever vectoriel en mode MMR (Maximum Marginal Relevance) avec BM25S : vectoriel_retriever = vectorstore.as_retriever(search_type="mmr", ...). Tu obtiens des résultats pertinents, diversifiés ET précis sur les mots-clés.
