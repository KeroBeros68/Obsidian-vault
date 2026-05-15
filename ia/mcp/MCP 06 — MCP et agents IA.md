#ia #mcp #agents #intégration #avancé

## MCP et agents IA

MCP et les agents IA sont conçus pour fonctionner ensemble. MCP standardise la couche d'outils des agents, rendant leur écosystème interopérable.

## La relation MCP ↔ Agents

```
Sans MCP :
  Agent LangChain → outils custom LangChain seulement
  Agent CrewAI    → outils custom CrewAI seulement
  → Chaque framework réinvente ses propres intégrations

Avec MCP :
  Agent LangChain ──[MCP]──> tous les serveurs MCP
  Agent CrewAI    ──[MCP]──> tous les serveurs MCP
  Agent AutoGen   ──[MCP]──> tous les serveurs MCP
  → Un serveur MCP = disponible pour tous les frameworks
```

> MCP est la couche d'outillage universelle des agents IA.

## Architecture agent + MCP

```
┌─────────────────────────────────────────────────┐
│                   AGENT IA                       │
│                                                  │
│  ┌──────────┐   ┌───────────┐   ┌────────────┐  │
│  │  LLM     │   │Orchestrat.│   │ MCP Client │  │
│  │(cerveau) │◄─►│ (boucle)  │◄─►│(connecteur)│  │
│  └──────────┘   └───────────┘   └─────┬──────┘  │
└───────────────────────────────────────│──────────┘
                                        │ MCP Protocol
                    ┌───────────────────┼────────────────┐
                    ↓                   ↓                ↓
             [Serveur           [Serveur          [Serveur
              Gmail MCP]         GitHub MCP]       DB MCP]
```

## Cas d'usage : agent de gestion de projet

```
Objectif : "Analyse les tickets GitHub cette semaine et envoie un résumé à l'équipe"

Agent planifie :
  1. Lister les issues GitHub de la semaine    → GitHub MCP
  2. Filtrer celles assignées à l'équipe       → GitHub MCP
  3. Calculer les métriques (ouvertes/fermées) → Tool calcul
  4. Rédiger le résumé                         → LLM
  5. Envoyer sur le canal Slack #équipe        → Slack MCP

Exécution :
  [GitHub MCP] → retourne 47 issues
  [GitHub MCP] → filtre → 23 issues équipe
  [Calcul]     → 15 fermées, 8 ouvertes, 3 bloquantes
  [LLM]        → rédige le résumé structuré
  [Slack MCP]  → envoie dans #équipe
  → TERMINÉ
```

## Construire un agent MCP avec LangChain

```python
from langchain_mcp_adapters.tools import load_mcp_tools
from langchain_anthropic import ChatAnthropic
from langgraph.prebuilt import create_react_agent
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
import asyncio

async def agent_mcp_complet():
    
    # Connecter plusieurs serveurs MCP
    github_params = StdioServerParameters(
        command="npx", args=["-y", "@modelcontextprotocol/server-github"],
        env={"GITHUB_TOKEN": "ghp_xxx"}
    )
    
    slack_params = StdioServerParameters(
        command="npx", args=["-y", "@modelcontextprotocol/server-slack"],
        env={"SLACK_BOT_TOKEN": "xoxb-xxx"}
    )
    
    # Charger les tools des deux serveurs
    all_tools = []
    
    async with stdio_client(github_params) as (r1, w1):
        async with ClientSession(r1, w1) as s1:
            await s1.initialize()
            all_tools += await load_mcp_tools(s1)
    
    async with stdio_client(slack_params) as (r2, w2):
        async with ClientSession(r2, w2) as s2:
            await s2.initialize()
            all_tools += await load_mcp_tools(s2)
    
    # Créer l'agent avec tous les tools MCP
    llm = ChatAnthropic(model="claude-sonnet-4-20250514")
    agent = create_react_agent(llm, all_tools)
    
    # Lancer la tâche
    résultat = await agent.ainvoke({
        "messages": [{
            "role": "user",
            "content": "Liste les 5 dernières issues GitHub du repo owner/repo et envoie un résumé sur le canal Slack #dev"
        }]
    })
    
    return résultat["messages"][-1].content
```

## MCP dans les systèmes multi-agents

```
[Agent Orchestrateur]
        ↓ délègue
        ├── [Agent Recherche]  ──[MCP]──> Brave Search MCP
        ├── [Agent Données]    ──[MCP]──> PostgreSQL MCP
        └── [Agent Rapport]    ──[MCP]──> Google Drive MCP
        ↓ synthétise
[Résultat final]
```

Chaque sous-agent peut avoir accès à ses propres serveurs MCP spécialisés.

## MCP vs outils custom : quand choisir quoi ?

| Situation | Recommandation |
|---|---|
| Service avec un serveur MCP officiel | ✅ Utilise le serveur MCP existant |
| Service populaire sans serveur MCP | ✅ Cherche un serveur communautaire sur mcp.so |
| Logique métier très spécifique à ton app | ✅ Crée un serveur MCP custom |
| Outil très simple et ponctuel | Tool custom LangChain peut suffire |
| Tu veux réutiliser l'outil avec plusieurs LLMs | ✅ Serveur MCP obligatoire |

## Serveurs MCP essentiels pour les agents

```
Accès données   : PostgreSQL, SQLite, MongoDB MCP
Fichiers        : Filesystem MCP, Google Drive MCP
Communication   : Gmail MCP, Slack MCP
Dev             : GitHub MCP, Docker MCP
Recherche       : Brave Search MCP, Perplexity MCP
Navigation web  : Puppeteer MCP, Playwright MCP
Calendrier      : Google Calendar MCP
```

> [!tip] MCP + Agentic RAG
> Un serveur MCP peut exposer un système RAG comme un Tool. L'agent appelle `recherche_documents(query)` via MCP → le serveur MCP gère toute la pipeline RAG → retourne les passages pertinents. C'est l'Agentic RAG clé en main.

> [!info] L'avenir : agents MCP natifs
> Anthropic développe des agents qui utilisent MCP nativement dès le départ, sans framework intermédiaire. Le protocole devient la colonne vertébrale de l'écosystème agent.
