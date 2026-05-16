#ia #langchain #prompts #templates #bases

## Prompt Templates

Les prompt templates sont des **moules réutilisables** avec des variables qu'on remplit à chaque appel.

## PromptTemplate — texte brut

```python
from langchain_core.prompts import PromptTemplate

# Template avec variable {sujet}
prompt = PromptTemplate.from_template(
    "Explique-moi {sujet} en 3 phrases simples."
)

# Remplir les variables → produit un StringPromptValue
résultat = prompt.invoke({"sujet": "la photosynthèse"})
print(résultat.text)
# → "Explique-moi la photosynthèse en 3 phrases simples."
```

## ChatPromptTemplate — messages structurés

Le plus utilisé avec les LLM modernes. Supporte les rôles system / human / assistant.

```python
from langchain_core.prompts import ChatPromptTemplate

# Template multi-variables et multi-rôles
prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es un expert en {domaine}. Réponds en {langue}."),
    ("human", "{question}")
])

# Remplir les variables
messages = prompt.invoke({
    "domaine": "astronomie",
    "langue": "français",
    "question": "C'est quoi un trou noir ?"
})
# → [SystemMessage(...), HumanMessage(...)]
```

## MessagesPlaceholder — injecter un historique

Permet d'injecter une liste de messages dynamique (historique de conversation).

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es un assistant utile."),
    # Ce placeholder sera remplacé par les messages précédents
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

# Injecter un historique
messages = prompt.invoke({
    "history": [
        ("human", "Mon prénom est Alice"),
        ("assistant", "Bonjour Alice !")
    ],
    "input": "Tu te souviens de mon prénom ?"
})
# → SystemMessage + HumanMessage(Alice) + AIMessage + HumanMessage(souviens?)
```

## Partial — pré-remplir des variables

Fige certaines variables à l'avance, laisse les autres libres.

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es un expert en {domaine}. Format : {format}"),
    ("human", "{question}")
])

# Figer le domaine et le format à l'avance
prompt_python = prompt.partial(
    domaine="Python",
    format="bullet points"
)

# Plus besoin de passer domaine et format
réponse = (prompt_python | llm).invoke({
    "question": "Quelles sont les bonnes pratiques ?"
})
```

## FewShotChatMessagePromptTemplate — exemples intégrés

Intègre des exemples dans le prompt (few-shot learning).

```python
from langchain_core.prompts import (
    ChatPromptTemplate,
    FewShotChatMessagePromptTemplate
)

# Exemples à montrer au LLM
exemples = [
    {"input": "2 + 2", "output": "4"},
    {"input": "10 × 5", "output": "50"},
    {"input": "100 ÷ 4", "output": "25"},
]

# Template pour chaque exemple
exemple_template = ChatPromptTemplate.from_messages([
    ("human", "{input}"),
    ("ai", "{output}")
])

# Few-shot prompt
few_shot_prompt = FewShotChatMessagePromptTemplate(
    example_prompt=exemple_template,
    examples=exemples
)

# Prompt complet avec les exemples intégrés
prompt_final = ChatPromptTemplate.from_messages([
    ("system", "Tu es une calculatrice. Réponds uniquement par le résultat."),
    few_shot_prompt,          # ← les exemples s'insèrent ici
    ("human", "{calcul}")
])

chain = prompt_final | llm | StrOutputParser()
print(chain.invoke({"calcul": "7 × 8"}))
# → "56"  (le LLM imite le pattern des exemples)
```

## Composer des prompts dynamiquement

```python
from langchain_core.prompts import ChatPromptTemplate

def créer_prompt_expert(domaine: str, langue: str = "français") -> ChatPromptTemplate:
    """Crée un prompt expert dynamiquement selon le domaine."""
    return ChatPromptTemplate.from_messages([
        ("system", f"""Tu es un expert en {domaine}.
Tu réponds toujours en {langue}.
Tu utilises des exemples concrets.
Tu adaptes ta réponse au niveau du public."""),
        ("human", "{question}")
    ])

# Créer des prompts spécialisés
prompt_médecin   = créer_prompt_expert("médecine")
prompt_juriste   = créer_prompt_expert("droit français")
prompt_dev       = créer_prompt_expert("développement Python")
```

## Récapitulatif

| Template | Usage |
|---|---|
| `PromptTemplate` | Texte brut simple, un seul rôle |
| `ChatPromptTemplate` | Messages multi-rôles, LLM de chat |
| `MessagesPlaceholder` | Injecter un historique de conversation |
| `FewShotChatMessagePromptTemplate` | Exemples intégrés dans le prompt |
| `.partial()` | Pré-remplir des variables fixes |

> [!tip] Toujours utiliser ChatPromptTemplate
> Pour tout LLM de type chat (Claude, GPT-4...), `ChatPromptTemplate` est toujours préférable à `PromptTemplate`. Il gère correctement les rôles et le format attendu.

> [!warning] Variables manquantes
> Si une variable du template n'est pas fournie lors de l'invoke, LangChain lève une `KeyError`. Utilise `.partial()` pour les variables fixes et évite les oublis.
