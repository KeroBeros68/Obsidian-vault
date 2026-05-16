#ia #langchain #multi-agents #langgraph #avancé

## Multi-agents avec LangGraph

## Architecture superviseur + sous-agents

```python
from langgraph.graph import StateGraph, END
from langgraph_supervisor import create_supervisor  # package séparé

# Agents spécialisés
agent_recherche = create_react_agent(llm, [outil_web], name="recherche")
agent_analyse   = create_react_agent(llm, [calculatrice], name="analyse")
agent_rédaction = create_react_agent(llm, [], name="rédaction")

# Superviseur qui coordonne
superviseur = create_supervisor(
    [agent_recherche, agent_analyse, agent_rédaction],
    model=llm,
    prompt="Tu coordonnes une équipe pour produire des rapports complets."
)

app = superviseur.compile()
résultat = app.invoke({"messages": [{"role": "user", "content": "Analyse le marché IA en 2025"}]})
```

## Sous-graphes — agents imbriqués

```python
from langgraph.graph import StateGraph, END

# Graphe enfant (sous-agent spécialisé)
def créer_sous_agent_recherche():
    graphe = StateGraph(State)
    graphe.add_node("chercher", nœud_recherche)
    graphe.add_node("filtrer", nœud_filtrage)
    graphe.add_edge("chercher", "filtrer")
    graphe.add_edge("filtrer", END)
    return graphe.compile()

sous_agent_recherche = créer_sous_agent_recherche()

# Graphe parent qui appelle le sous-graphe
def nœud_superviseur(state: State) -> dict:
    # Invoquer le sous-graphe comme une fonction normale
    résultat = sous_agent_recherche.invoke(state)
    return {"messages": résultat["messages"]}

graphe_principal = StateGraph(State)
graphe_principal.add_node("superviseur", nœud_superviseur)
graphe_principal.add_node("recherche", sous_agent_recherche)  # ← sous-graphe
```
