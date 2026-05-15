#ia #mcp #glossaire #référence

| Terme | Définition |
|---|---|
| **MCP** | Model Context Protocol. Protocole open-source créé par Anthropic (nov. 2024) pour standardiser la connexion entre LLM et outils/données externes. |
| **Host** | L'application qui intègre le LLM et gère les connexions aux serveurs MCP. Ex : Claude Desktop, Cursor, ton app custom. |
| **Client MCP** | Composant côté host qui maintient la connexion avec un serveur MCP et transmet les requêtes du LLM. |
| **Serveur MCP** | Programme qui expose des Tools, Resources et Prompts à n'importe quel client MCP compatible. |
| **Primitive** | L'un des 3 types de capacités qu'un serveur MCP peut exposer : Tool, Resource, ou Prompt. |
| **Tool** | Fonction que le LLM peut appeler pour agir dans le monde réel via MCP (envoyer un email, créer un ticket, lire une DB...). |
| **Resource** | Donnée en lecture seule exposée par un serveur MCP que le LLM peut consulter (fichier, document, données temps réel). |
| **Prompt (MCP)** | Template de prompt réutilisable exposé par un serveur MCP, encapsulant un workflow complexe en une commande simple. |
| **JSON-RPC 2.0** | Format de communication standard utilisé par MCP pour les messages entre client et serveur. |
| **stdio** | Mode de transport MCP où le serveur tourne en local et communique via entrée/sortie standard. Idéal pour les outils locaux. |
| **HTTP/SSE** | Mode de transport MCP où le serveur est distant (cloud) et communique via HTTP et Server-Sent Events. |
| **Server-Sent Events (SSE)** | Protocole de communication unidirectionnel serveur → client sur HTTP. Utilisé par MCP pour les connexions distantes. |
| **inputSchema** | La définition JSON Schema des paramètres attendus par un Tool MCP. Le LLM la lit pour savoir comment appeler le Tool. |
| **claude_desktop_config.json** | Fichier de configuration de Claude Desktop qui liste les serveurs MCP à connecter au démarrage. |
| **SDK MCP** | Kit de développement officiel pour créer des serveurs et clients MCP. Disponible en Python (`mcp`) et TypeScript (`@modelcontextprotocol/sdk`). |
| **Interopérabilité** | Capacité de MCP à connecter n'importe quel LLM à n'importe quel serveur MCP, indépendamment des frameworks utilisés. |
| **Sampling** | Capacité avancée MCP permettant au serveur de demander au LLM de générer du texte (LLM côté serveur). |
| **mcp.so** | Répertoire communautaire de serveurs MCP. |
| **modelcontextprotocol/servers** | Dépôt GitHub officiel des serveurs MCP maintenus par Anthropic (filesystem, GitHub, PostgreSQL, Slack, etc.). |
| **Tool poisoning** | Attaque de sécurité où un serveur MCP malveillant expose des Tools avec des descriptions trompeuses pour manipuler le LLM. |
| **Least privilege (MCP)** | Principe de sécurité : ne connecter que les serveurs MCP dont l'agent a réellement besoin, avec les permissions minimales. |
