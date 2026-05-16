#ia #langchain #retriever #rag #chain #pratique

## Retrievers et RAG Chain

## Retrievers — interface de recherche

```python
# Retriever basique depuis un vectorstore
retriever = vectorstore.as_retriever(
    search_type="similarity",   # ou "mmr"
    search_kwargs={"k": 4}
)

# Test manuel
docs = retriever.invoke("politique de retour")
print(f"{len(docs)} chunks récupérés")

# MMR — Maximum Marginal Relevance (résultats diversifiés)
retriever_mmr = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={"k": 4, "fetch_k": 20, "lambda_mult": 0.7}
)

# Avec filtre sur les métadonnées
retriever_filtré = vectorstore.as_retriever(
    search_kwargs={
        "k": 4,
        "filter": {"source": "politique.txt"}  # seulement ce fichier
    }
)
```

## MultiQueryRetriever — plusieurs reformulations

Génère plusieurs variantes de la question pour améliorer le recall.

```python
from langchain.retrievers import MultiQueryRetriever
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-sonnet-4-20250514")

retriever_multi = MultiQueryRetriever.from_llm(
    retriever=retriever,
    llm=llm
    # Pour "retour produit défectueux", génère aussi :
    # "procédure retour article endommagé"
    # "comment renvoyer un produit cassé"
    # → meilleure couverture sémantique
)
```

## ContextualCompressionRetriever — compresser les chunks

Récupère les chunks puis en extrait seulement les parties pertinentes.

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor

compresseur = LLMChainExtractor.from_llm(llm)

retriever_compressé = ContextualCompressionRetriever(
    base_compressor=compresseur,
    base_retriever=retriever
)
# → Retourne uniquement les phrases du chunk pertinentes à la question
```

## RAG Chain complète

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

prompt_rag = ChatPromptTemplate.from_messages([
    ("system", """Tu es un assistant expert.
Réponds UNIQUEMENT depuis le contexte fourni.
Si l'info est absente, dis-le clairement.
Cite toujours la source entre crochets.

Contexte :
{contexte}"""),
    ("human", "{question}")
])

def formater_docs(docs):
    return "\n\n---\n\n".join([
        f"[Source: {doc.metadata.get('source','?')} | Page: {doc.metadata.get('page','?')}]\n{doc.page_content}"
        for doc in docs
    ])

chain_rag = (
    {
        "contexte": retriever | formater_docs,
        "question": RunnablePassthrough()
    }
    | prompt_rag
    | llm
    | StrOutputParser()
)

réponse = chain_rag.invoke("Quelle est la politique de retour ?")
print(réponse)
```

## RAG avec sources — retourner les documents utilisés

```python
from langchain_core.runnables import RunnableParallel

# Retourner à la fois la réponse et les sources
chain_avec_sources = RunnableParallel(
    réponse=chain_rag,
    sources=retriever
)

résultat = chain_avec_sources.invoke("Politique de retour ?")
print(résultat["réponse"])
print("\nSources utilisées :")
for doc in résultat["sources"]:
    print(f"  - {doc.metadata.get('source')} p.{doc.metadata.get('page','?')}")
```

## RAG avec mémoire — chatbot documentaire

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory

prompt_rag_mémo = ChatPromptTemplate.from_messages([
    ("system", """Tu es un assistant documentaire.
Réponds depuis le contexte. Tiens compte de l'historique.

Contexte : {contexte}"""),
    MessagesPlaceholder("history"),
    ("human", "{question}")
])

chain_rag_mémo = (
    {
        "contexte": itemgetter("question") | retriever | formater_docs,
        "question": itemgetter("question"),
        "history": itemgetter("history")
    }
    | prompt_rag_mémo | llm | StrOutputParser()
)

sessions = {}
chain_finale = RunnableWithMessageHistory(
    chain_rag_mémo,
    lambda sid: sessions.setdefault(sid, InMemoryChatMessageHistory()),
    input_messages_key="question",
    history_messages_key="history"
)

config = {"configurable": {"session_id": "user1"}}
chain_finale.invoke({"question": "Quel est le délai de retour ?"}, config=config)
chain_finale.invoke({"question": "Et pour la garantie ?"}, config=config)
chain_finale.invoke({"question": "Résume ce qu'on a vu"}, config=config)
```

> [!tip] MultiQueryRetriever sur les questions vagues
> Quand les utilisateurs formulent mal leurs questions, `MultiQueryRetriever` améliore significativement le recall en générant plusieurs reformulations automatiquement.
