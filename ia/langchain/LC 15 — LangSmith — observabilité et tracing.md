#ia #langchain #langsmith #observabilité #tracing #production

## LangSmith — observabilité et tracing

LangSmith est la plateforme d'observabilité de LangChain. Elle enregistre chaque appel LLM, outil et étape de chain pour déboguer et améliorer tes applications.

## Configuration

```bash
pip install langsmith
```

```python
import os

# Variables d'environnement — ajouter dans .env
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "ta-clé-langsmith"
os.environ["LANGCHAIN_PROJECT"] = "mon-projet-rag"   # nom du projet

# Dès que ces vars sont définies, TOUT est automatiquement tracé
# Pas besoin de modifier ton code existant !
```

## Tracer automatiquement

```python
# Aucun changement de code nécessaire
# LangSmith intercepte automatiquement tous les appels LangChain

from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate

llm   = ChatAnthropic(model="claude-sonnet-4-20250514")
chain = ChatPromptTemplate.from_messages([("human", "{q}")]) | llm

# Cet appel est automatiquement tracé dans LangSmith
réponse = chain.invoke({"q": "Bonjour !"})
# → Visible dans https://smith.langchain.com/
```

## Tracer manuellement avec @traceable

```python
from langsmith import traceable

@traceable(name="pipeline_rag_complet")
def pipeline_rag(question: str) -> str:
    """Trace cette fonction entière comme un span LangSmith."""
    chunks = retriever.invoke(question)
    contexte = formater_docs(chunks)
    réponse = chain.invoke({"contexte": contexte, "question": question})
    return réponse

# L'appel apparaît comme un span de haut niveau dans LangSmith
résultat = pipeline_rag("Quelle est la politique de retour ?")
```

## Ajouter des métadonnées à une trace

```python
from langchain_core.runnables.config import RunnableConfig

config = RunnableConfig(
    metadata={
        "user_id": "alice_42",
        "session_id": "session_abc",
        "version_prompt": "v2.3",
        "environnement": "production"
    },
    tags=["rag", "support-client", "production"]
)

# Les métadonnées apparaissent dans LangSmith pour filtrer et analyser
réponse = chain.invoke({"question": "..."}, config=config)
```

## Évaluation dans LangSmith

```python
from langsmith import Client
from langsmith.evaluation import evaluate

client = Client()

# Définir des évaluateurs
def évaluateur_pertinence(run, example):
    """Note la pertinence de la réponse de 1 à 10."""
    réponse = run.outputs["output"]
    référence = example.outputs["réponse_attendue"]

    prompt = f"""Note la pertinence de 1 à 10.
Réponse générée : {réponse}
Réponse attendue : {référence}
Réponds uniquement par un chiffre."""

    score = float(llm.invoke(prompt).content.strip())
    return {"key": "pertinence", "score": score / 10}

# Lancer l'évaluation sur un dataset LangSmith
résultats = evaluate(
    lambda inputs: {"output": chain.invoke(inputs)},
    data="mon-dataset-eval",       # dataset créé dans LangSmith
    evaluators=[évaluateur_pertinence],
    experiment_prefix="rag-v2"
)
```

## Gestion des prompts avec LangSmith Hub

```python
from langsmith import Client

client = Client()

# Pousser un prompt dans LangSmith Hub
client.push_prompt(
    "support-client-v2",
    object=ChatPromptTemplate.from_messages([
        ("system", "Tu es un assistant support Acme Corp."),
        ("human", "{question}")
    ])
)

# Charger en production (toujours la dernière version)
from langchain import hub
prompt = hub.pull("mon-org/support-client-v2")

# Charger une version spécifique (rollback)
prompt = hub.pull("mon-org/support-client-v2:abc123")
```

## Métriques clés à surveiller dans LangSmith

```
Latence      : temps total et par étape (retrieval, LLM, parsing)
Tokens       : input/output par appel → coût
Erreurs      : taux d'erreur, types d'erreurs
Feedback     : scores utilisateurs (thumbs up/down)
Traces       : vue complète de chaque appel (ideal pour debug)
```

> [!tip] Activer le tracing dès le début
> Même en développement, active LangSmith dès le premier jour. Les traces de développement sont précieuses pour comprendre les comportements inattendus avant d'aller en production.

> [!info] Langfuse comme alternative open-source
> Si tu veux self-hoster l'observabilité, Langfuse est une excellente alternative open-source à LangSmith avec des fonctionnalités similaires.
