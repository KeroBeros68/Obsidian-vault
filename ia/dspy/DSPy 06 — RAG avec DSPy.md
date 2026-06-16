#ia #dspy #rag #retriever #pratique

## RAG avec DSPy

DSPy permet de construire des pipelines RAG modulaires et optimisables. La différence avec LangChain : les prompts du module de génération sont optimisés automatiquement par rapport à ta métrique.

## RAG minimal

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Signature RAG
class GenerateurRAG(dspy.Signature):
    """Génère une réponse à partir des passages récupérés.
    Ne réponds que depuis les passages fournis.
    Si l'information est absente, dis-le clairement.
    Cite toujours la source."""

    contexte: list[str] = dspy.InputField(desc="Passages récupérés de la base documentaire")
    question: str       = dspy.InputField(desc="Question de l'utilisateur")
    réponse:  str       = dspy.OutputField(desc="Réponse sourcée et précise")

# Module RAG
class RAGSimple(dspy.Module):
    def __init__(self, retriever):
        self.retriever  = retriever
        self.générateur = dspy.ChainOfThought(GenerateurRAG)

    def forward(self, question: str):
        # Récupérer les passages pertinents
        passages = self.retriever(question)

        # Générer la réponse
        réponse = self.générateur(contexte=passages, question=question)
        return réponse
```

## Intégration avec ChromaDB

```python
import dspy
import chromadb
from chromadb.utils import embedding_functions

# Configurer Chroma comme retriever
client     = chromadb.PersistentClient("./db")
ef         = embedding_functions.SentenceTransformerEmbeddingFunction(
    "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)
collection = client.get_or_create_collection("docs", embedding_function=ef)

def retriever_chroma(query: str, k: int = 4) -> list[str]:
    """Retriever Chroma compatible DSPy."""
    résultats = collection.query(query_texts=[query], n_results=k)
    return résultats["documents"][0]   # liste de strings

# Créer le pipeline RAG
rag = RAGSimple(retriever=retriever_chroma)

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Utiliser
résultat = rag(question="Quelle est la politique de retour ?")
print(résultat.réponse)
print(résultat.reasoning)   # raisonnement interne (ChainOfThought)
```

## RAG avancé avec raisonnement multi-hop

Pour des questions complexes qui nécessitent plusieurs recherches successives.

```python
import dspy

class QuestionDecomposeur(dspy.Signature):
    """Décompose une question complexe en sous-questions simples."""
    question:      str       = dspy.InputField()
    sous_questions: list[str] = dspy.OutputField(
        desc="2-3 sous-questions permettant de répondre à la question principale"
    )

class RAGMultiHop(dspy.Module):
    def __init__(self, retriever):
        self.retriever     = retriever
        self.décomposeur   = dspy.Predict(QuestionDecomposeur)
        self.générateur    = dspy.ChainOfThought(
            "contextes: list[str], question -> réponse"
        )

    def forward(self, question: str):
        # Décomposer la question
        décomposition = self.décomposeur(question=question)

        # Récupérer des passages pour chaque sous-question
        tous_les_contextes = []
        for sq in décomposition.sous_questions:
            passages = self.retriever(sq)
            tous_les_contextes.extend(passages)

        # Dédupliquer
        contextes_uniques = list(dict.fromkeys(tous_les_contextes))

        # Générer la réponse finale
        return self.générateur(contextes=contextes_uniques, question=question)
```

## RAG avec Hybrid Retriever (BM25S + vectoriel)

```python
import dspy
from langchain.retrievers import EnsembleRetriever

def créer_retriever_hybride(chunks, embeddings):
    from langchain_community.retrievers import BM25Retriever
    from langchain_community.vectorstores import Chroma

    bm25 = BM25Retriever.from_documents(chunks, k=4)
    vectorstore = Chroma.from_documents(chunks, embeddings)
    vectoriel = vectorstore.as_retriever(search_kwargs={"k": 4})

    return EnsembleRetriever(
        retrievers=[bm25, vectoriel],
        weights=[0.4, 0.6]
    )

def retriever_hybride_fn(query: str) -> list[str]:
    docs = hybrid_retriever.invoke(query)
    return [doc.page_content for doc in docs]

rag_hybride = RAGSimple(retriever=retriever_hybride_fn)
```

## RAG avec citation des sources

```python
import dspy

class RAGAvecSources(dspy.Signature):
    """Répond uniquement depuis les passages fournis et cite explicitement les sources."""

    passages:  list[dspy.InputField] = dspy.InputField(
        desc="Passages numérotés [1], [2], ... récupérés de la base documentaire"
    )
    question:  str = dspy.InputField()
    réponse:   str = dspy.OutputField(
        desc="Réponse avec citations inline sous forme [numéro]"
    )
    confiance: float = dspy.OutputField(
        desc="Score de confiance entre 0 et 1 selon la pertinence des passages"
    )

class RAGCitant(dspy.Module):
    def __init__(self, retriever):
        self.retriever = retriever
        self.générateur = dspy.ChainOfThought(RAGAvecSources)

    def forward(self, question: str):
        passages_bruts = self.retriever(question, k=5)

        # Numéroter les passages pour les citations
        passages_numérotés = [
            f"[{i+1}] {p}" for i, p in enumerate(passages_bruts)
        ]

        return self.générateur(passages=passages_numérotés, question=question)

rag = RAGCitant(retriever=mon_retriever)
résultat = rag(question="Comment retourner un produit défectueux ?")
print(résultat.réponse)   # → "Pour retourner un article défectueux [1], vous devez..."
print(résultat.confiance) # → 0.94
```

## Évaluer le RAG avant optimisation

```python
import dspy

# Dataset de test
dataset_test = [
    dspy.Example(
        question="Quel est le délai de retour ?",
        réponse_attendue="30 jours après réception"
    ).with_inputs("question"),
    dspy.Example(
        question="La livraison express coûte combien ?",
        réponse_attendue="9,90€"
    ).with_inputs("question"),
]

# Métrique simple
def métrique_rag(exemple, prédiction, trace=None) -> bool:
    réponse_attendue = exemple.réponse_attendue.lower()
    réponse_prédite  = prédiction.réponse.lower()
    return réponse_attendue in réponse_prédite

# Évaluer
évaluateur = dspy.Evaluate(
    devset=dataset_test,
    metric=métrique_rag,
    num_threads=4,
    display_progress=True
)

score_avant = évaluateur(rag)
print(f"Score avant optimisation : {score_avant:.1%}")

# Optimiser
optimiseur = dspy.BootstrapFewShot(metric=métrique_rag)
rag_optimisé = optimiseur.compile(rag, trainset=dataset_entraînement)

score_après = évaluateur(rag_optimisé)
print(f"Score après optimisation : {score_après:.1%}")
```

> [!tip] DSPy optimise le prompt du générateur, pas le retriever
> L'optimiseur DSPy améliore les instructions et les exemples few-shot du module de génération. Le retriever (Chroma, BM25S...) reste inchangé. Pour améliorer le retrieval, combine DSPy avec un bon retriever hybride.
