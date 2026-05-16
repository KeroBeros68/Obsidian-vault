#ia #langchain #chains #lcel #bases

## Chains et LCEL

LCEL (LangChain Expression Language) est le système de composition de LangChain. L'opérateur `|` connecte n'importe quelle brique qui implémente l'interface `Runnable`.

## L'interface Runnable

Toutes les briques LangChain implémentent la même interface :

```python
# Toute brique Runnable expose :
runnable.invoke(input)          # appel synchrone → 1 résultat
runnable.stream(input)          # appel streaming → chunks
runnable.batch(inputs)          # appel batch → liste de résultats
runnable.ainvoke(input)         # appel async → 1 résultat
runnable.astream(input)         # appel async streaming
runnable.abatch(inputs)         # appel async batch

# Ce sont : LLM, Prompt, Parser, Retriever, Tool, Chain...
# → tous chaînables avec |
```

## Chain basique

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_anthropic import ChatAnthropic

llm    = ChatAnthropic(model="claude-sonnet-4-20250514")
prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu résumes des textes en {nb} phrases."),
    ("human", "{texte}")
])
parser = StrOutputParser()

# Chaîner avec |
chain = prompt | llm | parser

# Invoquer avec un dict contenant les variables du template
résultat = chain.invoke({"nb": "2", "texte": "LangChain est un framework..."})
print(résultat)   # → string directement
```

## RunnablePassthrough — passer l'entrée intacte

```python
from langchain_core.runnables import RunnablePassthrough

# Utile pour passer des données en parallèle
chain = (
    {
        # "contexte" est traité par le retriever
        "contexte": retriever | formater_docs,
        # "question" est passée telle quelle
        "question": RunnablePassthrough()
    }
    | prompt
    | llm
    | StrOutputParser()
)
```

## RunnableLambda — fonction Python quelconque

```python
from langchain_core.runnables import RunnableLambda

def mettre_en_majuscules(texte: str) -> str:
    return texte.upper()

# Wrapper une fonction Python en Runnable
majuscule = RunnableLambda(mettre_en_majuscules)

chain = prompt | llm | StrOutputParser() | majuscule
# → La sortie sera en MAJUSCULES
```

## RunnableBranch — branchement conditionnel

```python
from langchain_core.runnables import RunnableBranch, RunnablePassthrough

# Chain pour les questions techniques
chain_technique = (
    ChatPromptTemplate.from_messages([("system", "Tu es un expert technique."), ("human", "{question}")])
    | llm | StrOutputParser()
)

# Chain pour les questions générales
chain_général = (
    ChatPromptTemplate.from_messages([("system", "Tu es un assistant général."), ("human", "{question}")])
    | llm | StrOutputParser()
)

# Classifier d'abord
chain_classif = (
    ChatPromptTemplate.from_messages([
        ("system", "Réponds uniquement 'TECH' ou 'GENERAL' selon le type de question."),
        ("human", "{question}")
    ])
    | llm | StrOutputParser()
)

# Branchement
branch = RunnableBranch(
    (lambda x: "TECH" in x["type"],    chain_technique),
    (lambda x: "GENERAL" in x["type"], chain_général),
    chain_général  # défaut
)

chain_complète = (
    {"question": RunnablePassthrough(), "type": {"question": RunnablePassthrough()} | chain_classif}
    | branch
)
```

## RunnableParallel — exécution en parallèle

```python
from langchain_core.runnables import RunnableParallel

# Deux chaînes exécutées simultanément
chain_résumé = prompt_résumé | llm | StrOutputParser()
chain_tags   = prompt_tags   | llm | StrOutputParser()

# Résultat : dict avec les deux sorties
parallel = RunnableParallel(
    résumé=chain_résumé,
    tags=chain_tags
)

résultat = parallel.invoke({"texte": "Article sur le RAG..."})
print(résultat["résumé"])  # → "Le RAG est une technique..."
print(résultat["tags"])    # → "RAG, LLM, LangChain"
```

## assign() — enrichir le dict au fil de la chain

```python
from langchain_core.runnables import RunnablePassthrough

chain = (
    RunnablePassthrough.assign(
        # Ajoute "résumé" au dict d'entrée
        résumé=lambda x: (prompt_résumé | llm | StrOutputParser()).invoke(x)
    )
    | RunnablePassthrough.assign(
        # Ajoute "traduction" basée sur le résumé
        traduction=lambda x: (prompt_trad | llm | StrOutputParser()).invoke({"texte": x["résumé"]})
    )
)

résultat = chain.invoke({"texte": "Article original..."})
print(résultat["texte"])       # → article original
print(résultat["résumé"])      # → résumé en français
print(résultat["traduction"])  # → résumé traduit en anglais
```

## itemgetter — extraire une clé du dict

```python
from operator import itemgetter

chain = (
    {"question": itemgetter("question"), "langue": itemgetter("langue")}
    | prompt
    | llm
    | StrOutputParser()
)

chain.invoke({"question": "Bonjour ?", "langue": "anglais", "autres_données": "ignorées"})
```

## Chain avec retry automatique

```python
# Retenter automatiquement en cas d'erreur API
chain_robuste = chain.with_retry(
    retry_if_exception_type=(Exception,),
    wait_exponential_jitter=True,  # délai exponentiel avec jitter
    stop_after_attempt=3           # max 3 tentatives
)
```

## Inspecter une chain

```python
# Voir le schéma d'entrée attendu
print(chain.input_schema.schema())
# → {"properties": {"domaine": {"type": "string"}, "question": {"type": "string"}}}

# Voir le schéma de sortie
print(chain.output_schema.schema())

# Afficher tous les steps
chain.steps   # liste des briques
```

> [!tip] Penser pipeline
> Une chain LCEL = un pipeline de transformations. Chaque `|` passe la sortie comme entrée de la brique suivante. Le dict d'entrée de `invoke()` doit contenir toutes les variables du premier template.

> [!warning] Les dicts dans les chains
> Quand tu écris `{"contexte": retriever | formater, "question": RunnablePassthrough()}`, tu crées un `RunnableParallel` implicite. Les deux branches s'exécutent en parallèle et leurs résultats sont combinés en un dict.
