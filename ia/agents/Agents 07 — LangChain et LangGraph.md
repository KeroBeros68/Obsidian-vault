#ia #agents #langchain #langgraph #frameworks #pratique

## LangChain et LangGraph

LangChain est le framework le plus utilisé pour construire des applications LLM. LangGraph en est l'extension pour les agents avec état complexe.

## LangChain — le couteau suisse

### Ce que LangChain apporte

```
Sans LangChain :
  Appel API OpenAI → parser la réponse → appeler l'outil → reformater → rappel API...
  → beaucoup de code boilerplate

Avec LangChain :
  chain = prompt | llm | output_parser
  result = chain.invoke({"question": "..."})
  → abstraction claire, modulaire
```

### Les concepts clés de LangChain

| Concept | Rôle |
|---|---|
| **Chain** | Séquence d'étapes enchaînées |
| **Prompt Template** | Template de prompt réutilisable |
| **LLM / ChatModel** | Interface unifiée vers tous les LLMs |
| **OutputParser** | Parse la sortie du LLM en structure Python |
| **Tool** | Outil que l'agent peut appeler |
| **Agent** | LLM + outils + boucle d'exécution |
| **Memory** | Gestion du contexte et de l'historique |
| **Retriever** | Interface vers une vector DB |

### Structure d'un agent LangChain simple

```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate

# 1. Définir les outils
@tool
def recherche_web(query: str) -> str:
    """Cherche des informations sur Internet."""
    # ... implémentation
    return résultats

# 2. Définir le LLM
llm = ChatOpenAI(model="gpt-4o")

# 3. Définir le prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es un assistant utile. Utilise les outils disponibles."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

# 4. Créer l'agent
agent = create_tool_calling_agent(llm, [recherche_web], prompt)
executor = AgentExecutor(agent=agent, tools=[recherche_web], verbose=True)

# 5. Exécuter
result = executor.invoke({"input": "Quel est le prix actuel du Bitcoin ?"})
```

---

## LangGraph — agents avec état complexe

LangGraph modélise l'agent comme un **graphe d'états**. Chaque nœud est une étape, chaque arête est une transition conditionnelle.

### Pourquoi LangGraph plutôt que LangChain ?

```
LangChain Agent  : boucle simple, difficile à contrôler finement
LangGraph        : graphe explicite, contrôle total sur les transitions
```

**Idéal pour** :
- Workflows complexes avec plusieurs chemins possibles
- Human-in-the-loop à des points précis
- État persistant entre les étapes
- Agents qui doivent parfois revenir en arrière

### Concepts LangGraph

```
État (State)     : dictionnaire partagé entre tous les nœuds
Nœud (Node)      : fonction Python qui modifie l'état
Arête (Edge)     : transition entre nœuds (fixe ou conditionnelle)
Point de contrôle: sauvegarde automatique de l'état (reprise possible)
```

### Structure d'un graphe LangGraph

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

# 1. Définir l'état
class AgentState(TypedDict):
    messages: list
    données: dict
    étape_courante: str

# 2. Définir les nœuds
def collecter_données(state: AgentState):
    # ... logique
    return {"données": résultats}

def analyser(state: AgentState):
    # ... logique
    return {"messages": [réponse]}

def doit_continuer(state: AgentState):
    if state["données"]["complet"]:
        return "analyser"
    return "collecter_données"  # boucle

# 3. Construire le graphe
graphe = StateGraph(AgentState)
graphe.add_node("collecter", collecter_données)
graphe.add_node("analyser", analyser)
graphe.add_conditional_edges("collecter", doit_continuer)
graphe.add_edge("analyser", END)
graphe.set_entry_point("collecter")

app = graphe.compile()
```

## Quand utiliser lequel

| Besoin | Outil |
|---|---|
| Agent simple avec quelques outils | LangChain AgentExecutor |
| Workflow avec branchements complexes | LangGraph |
| Human-in-the-loop précis | LangGraph |
| Pipeline RAG + agent | LangChain + LangGraph |
| Prototype rapide | LangChain |
| Production robuste | LangGraph |

> [!info] LangChain Hub
> LangChain Hub (hub.langchain.com) propose des centaines de prompts et chaînes prêts à l'emploi. Bon point de départ pour ne pas partir de zéro.

> [!tip] LangSmith pour le débogage
> LangSmith est l'outil de traçabilité de LangChain. Il permet de voir exactement ce qui se passe à chaque étape de ton agent (traces, latences, tokens). Indispensable en production.
