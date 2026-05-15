#ia #mcp #client #connexion #pratique

## Connecter un client MCP

Un client MCP peut être une application existante (Claude Desktop, Cursor) ou ton propre code Python/TypeScript.

## Option 1 — Claude Desktop (no-code)

La façon la plus simple de connecter des serveurs MCP à Claude.

### Installation

1. Télécharger Claude Desktop sur claude.ai/download
2. Ouvrir le fichier de configuration :
   - macOS : `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows : `%APPDATA%\Claude\claude_desktop_config.json`

### Configurer plusieurs serveurs

```json
{
  "mcpServers": {
    
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/moi/Documents"]
    },
    
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    },
    
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:password@localhost/ma_base"
      }
    },
    
    "mon-serveur-custom": {
      "command": "python",
      "args": ["/chemin/vers/mon_serveur.py"]
    }
  }
}
```

Redémarrer Claude Desktop → les outils apparaissent automatiquement.

---

## Option 2 — Client Python custom

Pour intégrer MCP dans ta propre application.

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
import asyncio

async def utiliser_serveur_mcp():
    
    # 1. Définir comment lancer le serveur
    params = StdioServerParameters(
        command="python",
        args=["mon_serveur.py"],
        env={"MA_CLE_API": "valeur"}
    )
    
    # 2. Se connecter au serveur
    async with stdio_client(params) as (read, write):
        async with ClientSession(read, write) as session:
            
            # 3. Initialiser la connexion
            await session.initialize()
            
            # 4. Lister les outils disponibles
            tools_result = await session.list_tools()
            print("Outils disponibles :")
            for tool in tools_result.tools:
                print(f"  - {tool.name}: {tool.description}")
            
            # 5. Appeler un outil
            résultat = await session.call_tool(
                "calculer",
                {"expression": "42 * 7 + 13"}
            )
            print(f"Résultat : {résultat.content[0].text}")
            
            # 6. Lister les ressources
            resources = await session.list_resources()
            for resource in resources.resources:
                print(f"  Resource: {resource.uri}")
            
            # 7. Lire une ressource
            contenu = await session.read_resource("file:///data/rapport.md")
            print(contenu)

asyncio.run(utiliser_serveur_mcp())
```

---

## Option 3 — Intégration avec LangChain

```python
from langchain_mcp_adapters.tools import load_mcp_tools
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def agent_avec_mcp():
    
    params = StdioServerParameters(
        command="python",
        args=["mon_serveur.py"]
    )
    
    async with stdio_client(params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            
            # Charger les tools MCP comme tools LangChain
            tools = await load_mcp_tools(session)
            
            # Créer un agent ReAct avec ces tools
            llm = ChatOpenAI(model="gpt-4o")
            agent = create_react_agent(llm, tools)
            
            # Utiliser l'agent
            résultat = await agent.ainvoke({
                "messages": "Calcule 144 * 7 puis dis-moi l'heure actuelle"
            })
            print(résultat["messages"][-1].content)

asyncio.run(agent_avec_mcp())
```

---

## Option 4 — Serveur HTTP/SSE (client distant)

Pour connecter un serveur MCP hébergé dans le cloud.

```python
from mcp.client.sse import sse_client
from mcp import ClientSession

async def connecter_serveur_distant():
    async with sse_client("https://mon-serveur-mcp.com/sse") as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools = await session.list_tools()
            # ... utilisation identique au client stdio
```

## Serveurs MCP prêts à installer

```bash
# Filesystem (accès aux fichiers locaux)
npx -y @modelcontextprotocol/server-filesystem ~/Documents

# GitHub
GITHUB_TOKEN=ghp_xxx npx -y @modelcontextprotocol/server-github

# PostgreSQL
npx -y @modelcontextprotocol/server-postgres postgresql://...

# Brave Search (recherche web)
BRAVE_API_KEY=xxx npx -y @modelcontextprotocol/server-brave-search

# Puppeteer (navigation web)
npx -y @modelcontextprotocol/server-puppeteer

# SQLite
npx -y @modelcontextprotocol/server-sqlite ma_base.db
```

> [!tip] Tester rapidement avec Claude Desktop
> C'est la façon la plus rapide de tester un serveur MCP. Configure-le dans le JSON, redémarre Claude Desktop, et teste directement dans la conversation.

> [!info] Répertoire officiel des serveurs
> github.com/modelcontextprotocol/servers contient les serveurs officiels maintenus par Anthropic. mcp.so liste les serveurs communautaires.
