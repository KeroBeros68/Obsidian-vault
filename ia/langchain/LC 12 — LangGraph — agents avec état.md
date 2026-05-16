#ia #langchain #langgraph #agents #état #avancé

## LangGraph — agents avec état complexe

LangGraph modélise un agent comme un **graphe d'états**. Chaque nœud est une étape, chaque arête est une transition — fixe ou conditionnelle.

## Pourquoi LangGraph plutôt que AgentExecutor

```
AgentExecutor (ancien) :
  Boucle fixe : LLM → outil → LLM → outil → ...
  Difficile de contrôler les transitions
  Pas de persistance d'état native

LangGraph :
  Graphe explicite avec états, nœuds, arêtes conditionnelles
  Contrôle total sur les transitions
  État persistant entre les nœuds
  Human-in-the-loop natif
  Recommandé pour les workflows complexes
```

## Concepts clés

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

# 1. L'État — dictionnaire partagé entre tous les nœuds
class AgentState(TypedDict):
    messages: Annotated[list, operator.add]  # les messages s'accumulent
    prochaine_étape: str
    données: dict
    nb_itérations: int

# 2. Les Nœuds — fonctions qui modifient l'état
def appeler_llm(state: AgentState) -> dict:
    """Appelle le LLM avec les messages actuels."""
    réponse = llm.invoke(state["messages"])
    return {"messages": [réponse]}

def exécuter_outil(state: AgentState) -> dict:
    """Exécute l'outil demandé par le LLM."""
    dernier_msg = state["messages"][-1]
    # Exécuter l'outil et ajouter le résultat
    résultat = exécuter(dernier_msg.tool_calls[0])
    return {"messages": [résultat]}

# 3. Les Arêtes conditionnelles — décident de la prochaine étape
def décider_suite(state: AgentState) -> str:
    dernier_msg = state["messages"][-1]
    if hasattr(dernier_msg, "tool_calls") and dernier_msg.tool_calls:
        return "exécuter_outil"   # → aller vers le nœud d'exécution
    return END                    # → terminer le graphe
```

## Agent ReAct custom avec LangGraph

```python
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from typing import TypedDict, Annotated
import operator

# ── Définir les outils ───────────────────────────────────
@tool
def calculatrice(expression: str) -> str:
    """Calcule une expression mathématique."""
    try:
        return str(eval(expression, {"__builtins__": {}}, {}))
    except Exception as e:
        return f"Erreur : {e}"

tools = [calculatrice]
llm = ChatAnthropic(model="claude-sonnet-4-20250514").bind_tools(tools)

# ── Définir l'état ────────────────────────────────────────
class State(TypedDict):
    messages: Annotated[list, operator.add]

# ── Définir les nœuds ─────────────────────────────────────
def nœud_llm(state: State) -> dict:
    réponse = llm.invoke(state["messages"])
    return {"messages": [réponse]}

# ToolNode gère automatiquement l'exécution des outils
nœud_outils = ToolNode(tools)

# ── Définir les arêtes ────────────────────────────────────
def doit_continuer(state: State) -> str:
    dernier_msg = state["messages"][-1]
    if hasattr(dernier_msg, "tool_calls") and dernier_msg.tool_calls:
        return "outils"
    return END

# ── Construire le graphe ──────────────────────────────────
graphe = StateGraph(State)

graphe.add_node("llm", nœud_llm)
graphe.add_node("outils", nœud_outils)

graphe.set_entry_point("llm")   # démarrer par le LLM

# Après le LLM : décider si on exécute un outil ou si on termine
graphe.add_conditional_edges(
    "llm",
    doit_continuer,
    {"outils": "outils", END: END}
)

# Après les outils : toujours repasser par le LLM
graphe.add_edge("outils", "llm")

# Compiler le graphe
app = graphe.compile()

# ── Utiliser le graphe ────────────────────────────────────
résultat = app.invoke({
    "messages": [{"role": "user", "content": "Calcule 15% de 1299 puis multiplie par 2"}]
})
print(résultat["messages"][-1].content)
```

## Human-in-the-Loop — validation humaine

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, END, interrupt

# Checkpointer pour sauvegarder l'état entre les interruptions
checkpointer = MemorySaver()

def nœud_avec_validation(state: State) -> dict:
    """S'arrête et attend une validation humaine."""
    action_proposée = générer_action(state)
    
    # interrupt() met en pause le graphe et retourne la valeur
    # L'exécution reprend quand on appelle app.invoke() à nouveau
    validation = interrupt({"action": action_proposée, "message": "Valider ?"})
    
    if validation == "oui":
        return exécuter_action(action_proposée)
    return {"messages": [{"role": "system", "content": "Action annulée."}]}

# Compiler avec checkpointer
app = graphe.compile(checkpointer=checkpointer)

config = {"configurable": {"thread_id": "session_1"}}

# Premier appel — s'arrête au interrupt()
résultat = app.invoke({"messages": [{"role": "user", "content": "..."}]}, config=config)
print("En attente de validation...")

# Reprendre après validation humaine
résultat_final = app.invoke(Command(resume="oui"), config=config)
```

## Graphe avec mémoire persistante

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.sqlite import SqliteSaver

# Mémoire in-memory (tests)
checkpointer = MemorySaver()

# Mémoire SQLite (persistante)
checkpointer = SqliteSaver.from_conn_string("./agent_state.db")

# Compiler avec persistance
app = graphe.compile(checkpointer=checkpointer)

config = {"configurable": {"thread_id": "alice_session"}}

# L'état est sauvegardé automatiquement après chaque nœud
app.invoke({"messages": [...]}, config=config)
# Redémarrer le programme...
# L'état est rechargé automatiquement au prochain invoke avec le même thread_id
```

## Visualiser le graphe

```python
# Afficher le graphe en ASCII
app.get_graph().print_ascii()

# Générer une image (nécessite pygraphviz)
from IPython.display import Image
Image(app.get_graph().draw_mermaid_png())
```

> [!tip] LangGraph vs create_react_agent
> `create_react_agent` est une factory qui construit un graphe LangGraph standard. Pour les cas simples, utilise-la. Pour les workflows complexes (branchements, human-in-the-loop, sous-graphes), construis le graphe manuellement.

> [!info] thread_id = session
> Le `thread_id` dans la config est l'identifiant de session. Même thread_id = même fil de conversation avec son état persistant. Différent thread_id = nouvelle conversation indépendante.
