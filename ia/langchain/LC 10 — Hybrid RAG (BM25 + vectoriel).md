#ia #langchain #rag #hybrid #bm25 #intermédiaire

## Hybrid RAG (BM25 + vectoriel)

Combine la recherche sémantique (vectorielle) et la recherche par mots-clés exacts (BM25) pour une meilleure couverture.

## Pourquoi combiner les deux

```
Vectoriel seul :
  "code erreur P0300" → peut trouver "problèmes moteur" (sens) mais pas "P0300" exact

BM25 seul :
  "panne de démarrage" → ne trouve pas "voiture ne démarre pas" (mots différents)

Hybrid :
  Les deux → précision sur les termes exacts + compréhension sémantique
```

## Installation

```bash
pip install rank-bm25
```

## BM25Retriever seul

```python
from langchain_community.retrievers import BM25Retriever

# Pas besoin d'embeddings ni de GPU
bm25_retriever = BM25Retriever.from_documents(chunks, k=4)

résultats = bm25_retriever.invoke("politique retour 30 jours")
for doc in résultats:
    print(doc.page_content[:150])
```

## EnsembleRetriever — fusion RRF

```python
from langchain_community.retrievers import BM25Retriever
from langchain.retrievers import EnsembleRetriever
from langchain_community.vectorstores import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings

# Retriever vectoriel
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)
vectorstore = Chroma.from_documents(chunks, embeddings, persist_directory="./db")
vectoriel = vectorstore.as_retriever(search_kwargs={"k": 4})

# Retriever BM25
bm25 = BM25Retriever.from_documents(chunks, k=4)

# Fusion avec pondération
hybrid = EnsembleRetriever(
    retrievers=[bm25, vectoriel],
    weights=[0.4, 0.6]   # BM25=40%, vectoriel=60%
)

résultats = hybrid.invoke("retour article défectueux 30 jours")
print(f"{len(résultats)} chunks fusionnés")
```

## RAG Chain avec Hybrid Retriever

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-sonnet-4-20250514")

prompt = ChatPromptTemplate.from_messages([
    ("system", """Tu es un assistant factuel.
Réponds uniquement depuis le contexte. Cite les sources.

Contexte : {contexte}"""),
    ("human", "{question}")
])

def formater_docs(docs):
    return "\n\n---\n\n".join([
        f"[{doc.metadata.get('source','?')}]\n{doc.page_content}"
        for doc in docs
    ])

chain_hybrid = (
    {"contexte": hybrid | formater_docs, "question": RunnablePassthrough()}
    | prompt | llm | StrOutputParser()
)

réponse = chain_hybrid.invoke("Quel est le délai de remboursement ?")
print(réponse)
```

## Tuner les poids selon le cas d'usage

```python
# Documentation technique (codes, noms propres)
hybrid_tech = EnsembleRetriever(
    retrievers=[bm25, vectoriel], weights=[0.6, 0.4]   # BM25 dominant
)

# FAQ / support client (questions vagues)
hybrid_support = EnsembleRetriever(
    retrievers=[bm25, vectoriel], weights=[0.3, 0.7]   # vectoriel dominant
)

# Contrats juridiques (termes exacts + contexte)
hybrid_legal = EnsembleRetriever(
    retrievers=[bm25, vectoriel], weights=[0.5, 0.5]   # équilibre
)
```

> [!tip] Hybrid = standard en production
> L'EnsembleRetriever avec BM25 + vectoriel donne systématiquement de meilleurs résultats que l'un ou l'autre seul. C'est l'approche recommandée pour les applications RAG en production.

> [!info] RRF interne
> LangChain utilise le Reciprocal Rank Fusion (RRF) en interne pour fusionner les deux listes de résultats. Les poids ajustent l'influence relative de chaque retriever dans le score final.
