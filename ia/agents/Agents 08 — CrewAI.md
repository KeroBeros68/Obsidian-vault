#ia #agents #crewai #multi-agents #pratique

## CrewAI

Framework multi-agents où l'on définit une **équipe d'agents avec des rôles** en langage naturel, et CrewAI gère la coordination. Le plus accessible pour débuter avec les multi-agents.

## Le concept

CrewAI s'inspire du travail en équipe humaine.

```
Equipe humaine :          Crew CrewAI :
  Chef de projet     →      Agent Orchestrateur
  Chercheur          →      Agent Recherche
  Rédacteur          →      Agent Rédaction
  Réviseur           →      Agent Éditorial

Chacun a un rôle, des objectifs, des outils
et sait quand passer le relais
```

## Les 4 concepts clés

### Agent
Un membre de l'équipe avec un rôle défini.

```python
from crewai import Agent

chercheur = Agent(
    role="Chercheur senior",
    goal="Trouver des informations précises et vérifiées sur le sujet donné",
    backstory="""Tu es un chercheur expérimenté avec 10 ans d'expérience
                 en veille stratégique. Tu es méticuleux et cite toujours
                 tes sources.""",
    tools=[outil_recherche_web, outil_rag],
    verbose=True
)
```

### Task (Tâche)
Ce qu'un agent doit accomplir.

```python
from crewai import Task

tâche_recherche = Task(
    description="""Recherche les dernières tendances en matière d'IA générative
                   en 2025. Focus sur les applications B2B. 
                   Produis un résumé structuré avec les 5 tendances principales.""",
    expected_output="Un résumé structuré en 5 points avec sources",
    agent=chercheur
)
```

### Crew (Équipe)
L'équipe et son mode de collaboration.

```python
from crewai import Crew, Process

équipe = Crew(
    agents=[chercheur, rédacteur, éditeur],
    tasks=[tâche_recherche, tâche_rédaction, tâche_révision],
    process=Process.sequential,  # ou hierarchical
    verbose=True
)

résultat = équipe.kickoff()
```

### Process (Mode de collaboration)

| Process | Description |
|---|---|
| `sequential` | Les tâches s'exécutent dans l'ordre, chacune reçoit le résultat de la précédente |
| `hierarchical` | Un manager LLM coordonne et délègue les tâches aux agents |

## Exemple complet — équipe de création de contenu

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool

outil_web = SerperDevTool()

# Agents
chercheur = Agent(
    role="Chercheur en tendances tech",
    goal="Identifier les tendances les plus importantes du sujet",
    backstory="Expert en veille technologique, synthèse et analyse.",
    tools=[outil_web]
)

rédacteur = Agent(
    role="Rédacteur de contenu tech",
    goal="Rédiger un article engageant basé sur la recherche",
    backstory="Rédacteur avec 5 ans d'expérience en contenu tech B2B.",
    tools=[]
)

# Tâches
recherche = Task(
    description="Recherche les 5 tendances majeures en IA en 2025",
    expected_output="Liste structurée de 5 tendances avec contexte",
    agent=chercheur
)

rédaction = Task(
    description="Écris un article de blog de 800 mots basé sur la recherche",
    expected_output="Article complet avec titre, intro, 5 sections, conclusion",
    agent=rédacteur
)

# Crew
crew = Crew(
    agents=[chercheur, rédacteur],
    tasks=[recherche, rédaction],
    process=Process.sequential
)

résultat = crew.kickoff(inputs={"sujet": "IA générative en entreprise"})
print(résultat)
```

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Très facile à prendre en main | Moins flexible que LangGraph |
| Rôles en langage naturel | Débogage plus difficile sur les workflows complexes |
| Bonne gestion de la collaboration | Coût élevé en tokens (chaque agent = un LLM call) |
| Idéal pour les workflows créatifs | Moins adapté aux workflows très techniques |

## Cas d'usage typiques

- ✅ Pipelines de création de contenu
- ✅ Équipes de recherche et synthèse
- ✅ Automatisation de workflows business (analyse, rapport, email)
- ✅ Prototypage rapide de systèmes multi-agents
- ❌ Workflows nécessitant un contrôle très fin sur les transitions d'état

> [!tip] Par où commencer
> Installe CrewAI (`pip install crewai crewai-tools`), définis 2 agents simples avec un processus séquentiel, et observe comment ils se passent le contexte. C'est la meilleure façon de comprendre le paradigme multi-agents en 30 minutes.
