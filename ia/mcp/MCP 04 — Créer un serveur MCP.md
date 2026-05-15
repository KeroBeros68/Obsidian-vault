#ia #mcp #serveur #python #typescript #pratique

## Créer un serveur MCP

Un serveur MCP expose des Tools, Resources et Prompts à n'importe quel client MCP compatible. Les SDK officiels simplifient grandement la création.

## SDK disponibles

| SDK | Langage | Installation |
|---|---|---|
| **MCP Python SDK** | Python | `pip install mcp` |
| **MCP TypeScript SDK** | TypeScript/JS | `npm install @modelcontextprotocol/sdk` |

## Serveur minimal en Python

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp import types
import asyncio

# 1. Créer l'instance du serveur
app = Server("mon-serveur-mcp")

# 2. Déclarer les outils disponibles
@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="calculer",
            description="Effectue un calcul mathématique simple. Utilise cet outil pour toute opération arithmétique.",
            inputSchema={
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "L'expression mathématique à calculer (ex: '2 + 2', '15 * 4 / 3')"
                    }
                },
                "required": ["expression"]
            }
        ),
        types.Tool(
            name="heure_actuelle",
            description="Retourne l'heure et la date actuelles. Utilise cet outil quand l'utilisateur demande l'heure ou la date.",
            inputSchema={
                "type": "object",
                "properties": {}
            }
        )
    ]

# 3. Implémenter l'exécution des outils
@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    
    if name == "calculer":
        try:
            expression = arguments["expression"]
            # Sécurisé : eval limité aux opérations mathématiques
            résultat = eval(expression, {"__builtins__": {}}, {})
            return [types.TextContent(
                type="text",
                text=f"Résultat de {expression} = {résultat}"
            )]
        except Exception as e:
            return [types.TextContent(
                type="text",
                text=f"Erreur de calcul : {str(e)}"
            )]
    
    elif name == "heure_actuelle":
        from datetime import datetime
        maintenant = datetime.now().strftime("%A %d %B %Y à %H:%M:%S")
        return [types.TextContent(
            type="text",
            text=f"Nous sommes le {maintenant}"
        )]
    
    else:
        raise ValueError(f"Outil inconnu : {name}")

# 4. Lancer le serveur en mode stdio
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

## Serveur avec Resources

```python
@app.list_resources()
async def list_resources() -> list[types.Resource]:
    return [
        types.Resource(
            uri="file:///data/rapport.md",
            name="Rapport mensuel",
            description="Le rapport d'activité du mois en cours",
            mimeType="text/markdown"
        )
    ]

@app.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "file:///data/rapport.md":
        with open("/data/rapport.md", "r") as f:
            return f.read()
    raise ValueError(f"Ressource inconnue : {uri}")
```

## Serveur en TypeScript (équivalent)

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { ListToolsRequestSchema, CallToolRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "mon-serveur-mcp", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "calculer",
    description: "Effectue un calcul mathématique.",
    inputSchema: {
      type: "object",
      properties: {
        expression: { type: "string", description: "Expression à calculer" }
      },
      required: ["expression"]
    }
  }]
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "calculer") {
    const { expression } = request.params.arguments as { expression: string };
    const résultat = eval(expression);
    return { content: [{ type: "text", text: `${expression} = ${résultat}` }] };
  }
  throw new Error(`Outil inconnu : ${request.params.name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Configurer le serveur dans Claude Desktop

Ajouter dans `claude_desktop_config.json` :

```json
{
  "mcpServers": {
    "mon-serveur": {
      "command": "python",
      "args": ["/chemin/vers/mon_serveur.py"],
      "env": {
        "MA_CLE_API": "valeur"
      }
    }
  }
}
```

Emplacement du fichier de config :
- macOS : `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows : `%APPDATA%\Claude\claude_desktop_config.json`

> [!tip] Serveurs MCP prêts à l'emploi
> Avant de créer ton propre serveur, vérifie si un serveur officiel ou communautaire existe déjà pour ton service sur github.com/modelcontextprotocol/servers ou sur mcp.so (répertoire communautaire).

> [!warning] Ne jamais hardcoder les credentials
> Utilise toujours les variables d'environnement (`env` dans la config) pour les clés API, mots de passe et tokens. Ne les mets jamais directement dans le code.
