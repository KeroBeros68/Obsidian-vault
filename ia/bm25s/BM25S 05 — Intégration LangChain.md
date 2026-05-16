#ia #bm25s #langchain #retriever #intégration #pratique

## Intégration LangChain

Le `BM25Retriever` natif de LangChain utilise `rank_bm25` sous le capot — lent et sans persistance. Voici comment créer un retriever LangChain propulsé par bm25s.

## Le retriever custom BM25SRetriever

```python
import bm25s
import Stemmer
from langchain_core.retrievers import BaseRetriever
from langchain_core.documents import Document
from langchain_core.callbacks import CallbackManagerForRetrieverRun
from pydantic import Field
from typing import List, Optional
import os

class BM25SRetriever(BaseRetriever):
    """
    Retriever LangChain propulsé par bm25s.
    500× plus rapide que rank-bm25, avec persistance et stemming.
    """

    bm25_index: bm25s.BM25 = Field(description="Index bm25s")
    documents: List[Document] = Field(description="Documents LangChain originaux")
    k: int = Field(default=4, description="Nombre de résultats à retourner")
    stemmer: Optional[object] = Field(default=None, description="Stemmer PyStemmer")

    class Config:
        arbitrary_types_allowed = True   # autorise les types non-Pydantic

    # ── Constructeurs ─────────────────────────────────────

    @classmethod
    def from_documents(
        cls,
        documents: List[Document],
        k: int = 4,
        langue: str = "french",
        avec_stemming: bool = True,
        index_path: Optional[str] = None   # chemin de sauvegarde optionnel
    ) -> "BM25SRetriever":
        """Crée un BM25SRetriever depuis une liste de Documents LangChain."""

        stemmer = Stemmer.Stemmer(langue) if avec_stemming else None
        textes  = [doc.page_content for doc in documents]

        corpus_tokenisé = bm25s.tokenize(
            textes,
            stemmer=stemmer,
            stopwords=langue if langue in ["french", "english", "german", "spanish"] else None
        )

        index = bm25s.BM25()
        index.index(corpus_tokenisé)

        # Sauvegarder si un chemin est fourni
        if index_path:
            index.save(index_path, corpus=textes)

        return cls(bm25_index=index, documents=documents, k=k, stemmer=stemmer)

    @classmethod
    def from_index(
        cls,
        index_path: str,
        documents: List[Document],
        k: int = 4,
        langue: str = "french"
    ) -> "BM25SRetriever":
        """Recharge un BM25SRetriever depuis un index sauvegardé."""
        index   = bm25s.BM25.load(index_path, load_corpus=False)
        stemmer = Stemmer.Stemmer(langue)
        return cls(bm25_index=index, documents=documents, k=k, stemmer=stemmer)

    # ── Méthode principale ────────────────────────────────

    def _get_relevant_documents(
        self,
        query: str,
        *,
        run_manager: CallbackManagerForRetrieverRun
    ) -> List[Document]:
        """Recherche BM25S → retourne des Documents LangChain avec score."""

        query_tokenisée = bm25s.tokenize(
            [query],
            stemmer=self.stemmer,
            stopwords="french" if self.stemmer else None
        )

        n = min(self.k, len(self.documents))
        résultats_idx, scores = self.bm25_index.retrieve(query_tokenisée, k=n)

        docs_trouvés = []
        for i in range(résultats_idx.shape[1]):
            idx   = int(résultats_idx[0, i])
            score = float(scores[0, i])
            doc   = self.documents[idx]

            # Ajouter le score BM25S dans les métadonnées
            doc_enrichi = Document(
                page_content=doc.page_content,
                metadata={**doc.metadata, "bm25s_score": round(score, 4)}
            )
            docs_trouvés.append(doc_enrichi)

        return docs_trouvés
```

## Utilisation simple

```python
from langchain_community.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Charger et découper les documents
loader = TextLoader("politique.txt", encoding="utf-8")
docs   = loader.load()
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=50)
chunks = splitter.split_documents(docs)

# Créer le retriever BM25S
retriever = BM25SRetriever.from_documents(
    chunks,
    k=4,
    langue="french",
    avec_stemming=True,
    index_path="./index_bm25s"   # sauvegarde automatique
)

# Utiliser comme n'importe quel retriever LangChain
résultats = retriever.invoke("retour produit défectueux")
for doc in résultats:
    print(f"Score BM25S : {doc.metadata['bm25s_score']}")
    print(f"Source      : {doc.metadata.get('source', '?')}")
    print(f"Texte       : {doc.page_content[:150]}\n")
```

## Intégration dans une RAG Chain

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
        f"[{doc.metadata.get('source','?')} | BM25={doc.metadata.get('bm25s_score',0):.2f}]\n{doc.page_content}"
        for doc in docs
    ])

# Chain RAG avec BM25S
chain_bm25s = (
    {"contexte": retriever | formater_docs, "question": RunnablePassthrough()}
    | prompt | llm | StrOutputParser()
)

réponse = chain_bm25s.invoke("Quel est le délai de remboursement ?")
print(réponse)
```

## Rechargement de l'index (session suivante)

```python
# Session suivante — charger depuis le disque sans reconstruire
retriever_rechargé = BM25SRetriever.from_index(
    index_path="./index_bm25s",
    documents=chunks,   # passer les Documents LangChain
    k=4,
    langue="french"
)

# Utilisation identique
résultats = retriever_rechargé.invoke("garantie produit")
```

> [!tip] Sauvegarder l'index en production
> Passe toujours `index_path` à `from_documents()` en production. L'index se reconstruit en quelques secondes sur de petits corpus, mais peut prendre plusieurs minutes sur de grands corpus. La sauvegarde évite ce coût au redémarrage.

> [!warning] Reconstruire en cas de mise à jour du corpus
> BM25S ne supporte pas l'ajout incrémental de documents. Si ton corpus évolue, supprime l'ancien index et appelle à nouveau `from_documents()` avec le nouveau corpus complet.
