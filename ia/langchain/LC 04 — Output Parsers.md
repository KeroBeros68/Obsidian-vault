#ia #langchain #output-parsers #bases

## Output Parsers

Transforment la sortie brute du LLM (AIMessage) en types Python utilisables : string, dict, objet Pydantic, liste...

## Vue d'ensemble

```
LLM retourne toujours un AIMessage
        ↓ Output Parser
string      → StrOutputParser
dict        → JsonOutputParser
objet typé  → PydanticOutputParser
liste       → CommaSeparatedListOutputParser
bool        → BooleanOutputParser
```

## StrOutputParser — le plus simple

```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()

# AIMessage → string propre
texte = parser.invoke(réponse_llm)
print(type(texte))   # → <class 'str'>

# Dans une chain
chain = prompt | llm | StrOutputParser()
résultat = chain.invoke({"question": "Bonjour ?"})
print(type(résultat))  # → str directement
```

## JsonOutputParser — pour obtenir un dict

```python
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.prompts import ChatPromptTemplate

parser = JsonOutputParser()

prompt = ChatPromptTemplate.from_messages([
    ("system", """Réponds UNIQUEMENT en JSON valide.
                  Pas de markdown, pas d'explication, juste le JSON.
                  Format : {{"nom": "...", "age": ..., "ville": "..."}}"""),
    ("human", "Extrait les infos : {texte}")
])

chain = prompt | llm | parser

résultat = chain.invoke({
    "texte": "Marie Dupont, 34 ans, habite Lyon."
})

print(type(résultat))       # → dict
print(résultat["nom"])      # → "Marie Dupont"
print(résultat["age"])      # → 34
```

## PydanticOutputParser — sortie typée et validée

```python
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field
from typing import List

class ArticleBlog(BaseModel):
    titre: str = Field(description="Titre accrocheur de l'article")
    résumé: str = Field(description="Résumé en 2 phrases")
    tags: List[str] = Field(description="3 à 5 tags pertinents")
    temps_lecture: int = Field(description="Temps de lecture estimé en minutes")
    niveau: str = Field(description="débutant, intermédiaire ou avancé")

parser = PydanticOutputParser(pydantic_object=ArticleBlog)

prompt = ChatPromptTemplate.from_messages([
    ("system", """Tu génères des métadonnées d'articles de blog.
{format_instructions}"""),
    ("human", "Génère les métadonnées pour un article sur : {sujet}")
])

# Injecter automatiquement les instructions de format dans le prompt
prompt_formaté = prompt.partial(
    format_instructions=parser.get_format_instructions()
)

chain = prompt_formaté | llm | parser

article = chain.invoke({"sujet": "Introduction au RAG avec LangChain"})

print(type(article))           # → <class 'ArticleBlog'>
print(article.titre)           # → "Construire un RAG avec LangChain..."
print(article.tags)            # → ["RAG", "LangChain", "Python", "LLM"]
print(article.temps_lecture)   # → 8
print(article.niveau)          # → "intermédiaire"
```

## CommaSeparatedListOutputParser

```python
from langchain_core.output_parsers import CommaSeparatedListOutputParser

parser = CommaSeparatedListOutputParser()

prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu listes des éléments séparés par des virgules, rien d'autre."),
    ("human", "Liste 5 frameworks Python populaires")
])

chain = prompt | llm | parser
résultat = chain.invoke({})

print(type(résultat))   # → list
print(résultat)         # → ["Django", "FastAPI", "Flask", "LangChain", "Pydantic"]
```

## XMLOutputParser — pour du XML structuré

```python
from langchain_core.output_parsers import XMLOutputParser

parser = XMLOutputParser(tags=["produit", "nom", "prix", "disponible"])

prompt = ChatPromptTemplate.from_messages([
    ("system", "Réponds en XML structuré avec les balises : produit, nom, prix, disponible"),
    ("human", "{description}")
])

chain = prompt | llm | parser
résultat = chain.invoke({"description": "iPhone 15 Pro, 1299€, en stock"})
# → {"produit": {"nom": "iPhone 15 Pro", "prix": "1299", "disponible": "true"}}
```

## OutputFixingParser — retry automatique

> [!warning] Dépréciation prévue
> `from langchain.output_parsers import OutputFixingParser` fonctionne encore mais sera migré vers `langchain_community`. Suivre les release notes LangChain pour la migration.

Si le LLM retourne un JSON malformé, ce parser relance un appel pour corriger.

```python
from langchain.output_parsers import OutputFixingParser
from langchain_core.output_parsers import PydanticOutputParser

# Parser de base
parser_base = PydanticOutputParser(pydantic_object=ArticleBlog)

# Parser avec auto-correction
parser_robuste = OutputFixingParser.from_llm(
    parser=parser_base,
    llm=llm   # LLM utilisé pour corriger les sorties invalides
)

# Si le LLM retourne un JSON invalide :
# 1. PydanticOutputParser échoue à parser
# 2. OutputFixingParser relance le LLM avec le texte invalide + instructions
# 3. Le LLM corrige le format
# 4. Re-tentative de parsing
chain_robuste = prompt_formaté | llm | parser_robuste
```

## RetryOutputParser — retry avec le prompt

```python
from langchain.output_parsers import RetryOutputParser

parser_retry = RetryOutputParser.from_llm(
    parser=parser_base,
    llm=llm,
    max_retries=3
)
# Relance jusqu'à 3 fois avec le prompt original si le parsing échoue
```

## Choisir le bon parser

| Besoin | Parser |
|---|---|
| Réponse textuelle simple | `StrOutputParser` |
| Dict Python depuis JSON | `JsonOutputParser` |
| Objet Python typé et validé | `PydanticOutputParser` |
| Liste d'éléments | `CommaSeparatedListOutputParser` |
| XML structuré | `XMLOutputParser` |
| Robustesse avec auto-correction | `OutputFixingParser` |
| Sortie structurée garantie | `llm.with_structured_output()` |

> [!tip] with_structured_output() > PydanticOutputParser
> `llm.with_structured_output(MonSchema)` est plus fiable car le format est contraint au niveau de l'API (function calling), pas juste dans le prompt. Utilise-le quand le LLM le supporte.

> [!warning] JsonOutputParser peut échouer
> Si le LLM ajoute du markdown (```json ... ```) autour de son JSON, le parser échoue. Soit tu instrus très clairement "sans markdown", soit tu utilises `OutputFixingParser` pour la robustesse.
