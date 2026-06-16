#ia #langchain #agents #tools #react #intermédiaire

## Tools et agents ReAct

Un agent ReAct utilise un LLM pour décider quels outils appeler, dans quel ordre, jusqu'à atteindre son objectif.

## Définir des outils avec @tool

```python
from langchain_core.tools import tool

@tool
def calculatrice(expression: str) -> str:
    """Calcule une expression mathématique.
    Utilise cet outil pour toute opération arithmétique.
    Exemples valides : '2 + 2', '15 * 4 / 3', '(100 - 20) * 1.2'
    """
    try:
        résultat = eval(expression, {"__builtins__": {}}, {})
        return f"{expression} = {résultat}"
    except Exception as e:
        return f"Erreur de calcul : {str(e)}"

@tool
def longueur_texte(texte: str) -> str:
    """Compte le nombre de mots et de caractères dans un texte."""
    mots = len(texte.split())
    chars = len(texte)
    return f"{mots} mots, {chars} caractères"

@tool
def date_actuelle() -> str:
    """Retourne la date et l'heure actuelles.
    Utilise cet outil quand l'utilisateur demande la date ou l'heure.
    """
    from datetime import datetime
    return datetime.now().strftime("%A %d %B %Y à %H:%M")

# Inspecter un tool
print(calculatrice.name)          # → "calculatrice"
print(calculatrice.description)   # → la docstring
print(calculatrice.args)          # → {"expression": {"type": "string"}}
```

## StructuredTool — pour des paramètres complexes

```python
from langchain_core.tools import StructuredTool
from pydantic import BaseModel, Field

class RechercheProduitInput(BaseModel):
    nom: str = Field(description="Nom du produit à rechercher")
    catégorie: str = Field(description="Catégorie : électronique, vêtements, alimentaire")
    prix_max: float = Field(description="Prix maximum en euros", default=1000.0)

def rechercher_produit(nom: str, catégorie: str, prix_max: float = 1000.0) -> str:
    """Recherche un produit dans le catalogue."""
    return f"Trouvé : {nom} en {catégorie} à moins de {prix_max}€"

outil_recherche = StructuredTool.from_function(
    func=rechercher_produit,
    name="recherche_produit",
    description="Recherche un produit dans le catalogue de la boutique.",
    args_schema=RechercheProduitInput
)
```

## Créer un agent ReAct avec LangGraph

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent

llm = ChatAnthropic(model="claude-sonnet-4-20250514")

@tool
def calculatrice(expression: str) -> str:
    """Calcule une expression mathématique."""
    try:
        return str(eval(expression, {"__builtins__": {}}, {}))
    except Exception as e:
        return f"Erreur : {e}"

@tool
def recherche_web(query: str) -> str:
    """Cherche des informations sur Internet.
    Utilise pour les actualités, prix temps réel, infos récentes.
    """
    # Simulé ici — en vrai : Tavily, SerpAPI, Brave Search...
    return f"Résultats pour '{query}': [résultats simulés]"

tools = [calculatrice, recherche_web]

# Créer l'agent ReAct
agent = create_react_agent(llm, tools)

# Invoquer l'agent
résultat = agent.invoke({
    "messages": [{"role": "user", "content": "Combien font 123 × 456 ?"}]
})

# La réponse finale est le dernier message
print(résultat["messages"][-1].content)
```

## Agent avec system prompt personnalisé

```python
from langchain_core.messages import SystemMessage

system_prompt = SystemMessage(content="""Tu es un assistant expert en analyse financière.
Tu utilises les outils disponibles pour répondre précisément.
Tu présentes toujours tes calculs de manière claire.
Tu signales quand une information est incertaine.""")

agent = create_react_agent(
    llm,
    tools,
    state_modifier=system_prompt   # injecté comme message système
)
```

## Voir le raisonnement de l'agent

```python
# stream() montre chaque étape du raisonnement
for étape in agent.stream(
    {"messages": [{"role": "user", "content": "Calcule 15% de 1299€ puis dis-moi la date"}]},
    stream_mode="values"
):
    dernier_msg = étape["messages"][-1]
    dernier_msg.pretty_print()
    # → Affiche : "Thought", "Action: calculatrice(1299*0.15)", "Observation: 194.85"
    # → Puis    : "Action: date_actuelle()", "Observation: jeudi 15 mai 2025..."
    # → Enfin   : "Réponse: 15% de 1299€ = 194,85€. Nous sommes le jeudi..."
```

## Outils avec état — accès à des ressources externes

```python
from langchain_community.tools import DuckDuckGoSearchRun
from langchain_community.tools.tavily_search import TavilySearchResults

# Recherche web réelle avec DuckDuckGo (gratuit)
tool_ddg = DuckDuckGoSearchRun()

# Recherche web avec Tavily (meilleure qualité, nécessite API key)
tool_tavily = TavilySearchResults(max_results=3)

# Calculatrice
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
tool_wiki = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())

# Agent avec tools réels
agent_réel = create_react_agent(llm, [tool_ddg, tool_wiki])
```

## Limiter les itérations de l'agent

```python
from langgraph.prebuilt import create_react_agent

agent = create_react_agent(
    llm,
    tools,
    # Limiter à 10 itérations maximum
)

# Avec timeout via invoke config
résultat = agent.invoke(
    {"messages": [{"role": "user", "content": "Ma question"}]},
    config={"recursion_limit": 10}   # max 10 appels LLM
)
```

> [!tip] La docstring est le prompt de l'outil
> Le LLM décide quel outil appeler uniquement en lisant sa description. Soigne chaque docstring : indique quand utiliser l'outil, quand ne pas l'utiliser, et des exemples de paramètres valides.

> [!warning] Toujours définir recursion_limit
> Sans limite, un agent mal configuré peut boucler indéfiniment et consommer des tokens à l'infini. `recursion_limit=10` est un bon défaut pour commencer.
