#ia #langchain #bases #définition

## Qu'est-ce que LangChain ?

Framework Python/JavaScript open-source qui fournit des **abstractions standardisées** pour construire des applications basées sur des LLM : chatbots, agents, pipelines RAG, automatisations.

## La philosophie — des Lego

```
Sans LangChain :
  Appel API brut → parser la réponse → reformater → rappel API
  → beaucoup de code boilerplate, rien de réutilisable

Avec LangChain :
  prompt | llm | parser
  → briques standardisées, connectables avec |
  → chaque brique est testable et remplaçable indépendamment
```

## L'opérateur | — le pipe LCEL

LCEL = LangChain Expression Language. Le `|` connecte les briques comme un pipeline Unix.

```python
# La sortie de chaque brique devient l'entrée de la suivante
chain = prompt | llm | output_parser

# Équivalent à :
# 1. messages = prompt.invoke(input)
# 2. réponse  = llm.invoke(messages)
# 3. résultat = output_parser.invoke(réponse)
```

## Les 5 briques fondamentales

```
┌──────────────────────────────────────────────┐
│              LANGCHAIN                        │
│                                              │
│  ┌────────────┐  → formate l'entrée          │
│  │  Prompt    │                              │
│  │  Template  │                              │
│  └────────────┘                              │
│        ↓ |                                   │
│  ┌────────────┐  → génère la réponse         │
│  │    LLM     │                              │
│  │  ChatModel │                              │
│  └────────────┘                              │
│        ↓ |                                   │
│  ┌────────────┐  → transforme la sortie      │
│  │  Output    │                              │
│  │  Parser    │                              │
│  └────────────┘                              │
│        ↓ |                                   │
│  ┌────────────┐  → stocke/cherche des docs   │
│  │  Retriever │                              │
│  └────────────┘                              │
│        ↓ |                                   │
│  ┌────────────┐  → agit dans le monde réel   │
│  │   Tools    │                              │
│  │  + Agents  │                              │
│  └────────────┘                              │
└──────────────────────────────────────────────┘
```

## Installation

```bash
# Core LangChain
pip install langchain langchain-core

# Intégrations LLM
pip install langchain-anthropic   # Claude
pip install langchain-openai      # GPT + embeddings OpenAI

# Intégrations communautaires (retrievers, loaders, vectorstores...)
pip install langchain-community

# LangGraph (agents avec état)
pip install langgraph

# Vectorstore local
pip install chromadb

# Embeddings open-source
pip install sentence-transformers
```

## Structure du package LangChain

```
langchain-core        : interfaces de base (Runnable, BasePrompt...)
langchain             : chaînes, agents, mémoire de haut niveau
langchain-community   : intégrations tierces (loaders, vectorstores...)
langchain-anthropic   : intégration Claude spécifique
langchain-openai      : intégration OpenAI spécifique
langgraph             : agents avec graphe d'états
langserve             : déploiement en API REST
langsmith             : observabilité et tracing
```

## Premier programme complet

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 3 briques
llm    = ChatAnthropic(model="claude-sonnet-4-20250514")
prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es un expert en {domaine}."),
    ("human", "{question}")
])
parser = StrOutputParser()

# 1 chain
chain = prompt | llm | parser

# 1 appel
réponse = chain.invoke({
    "domaine": "astronomie",
    "question": "Combien y a-t-il d'étoiles dans la Voie Lactée ?"
})
print(réponse)
```

> [!tip] Mémo
> LangChain = Lego pour LLM. Chaque brique est indépendante, testable, et remplaçable. L'opérateur `|` les connecte en pipeline.

> [!info] LangChain vs LlamaIndex
> LangChain = couteau suisse, bon pour tout (agents, RAG, chaînes). LlamaIndex = spécialiste des données et du RAG complexe. Les deux sont complémentaires.
