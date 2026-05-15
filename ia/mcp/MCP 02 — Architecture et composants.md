#ia #mcp #architecture #composants #bases

## Architecture et composants MCP

MCP repose sur une architecture **client-serveur** simple et standardisée.

## Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────┐
│                     APPLICATION HÔTE                         │
│  (Claude Desktop, Cursor, ton app custom...)                 │
│                                                              │
│   ┌──────────────┐         ┌──────────────────────────────┐ │
│   │   LLM        │◄───────►│      CLIENT MCP              │ │
│   │  (Claude,    │         │  (gère les connexions        │ │
│   │   GPT-4...)  │         │   aux serveurs)              │ │
│   └──────────────┘         └──────────────┬───────────────┘ │
└──────────────────────────────────────────-│-────────────────┘
                                            │ Protocole MCP
                          ┌─────────────────┼────────────────┐
                          ↓                 ↓                ↓
                   ┌──────────┐    ┌──────────┐    ┌──────────┐
                   │ Serveur  │    │ Serveur  │    │ Serveur  │
                   │  Gmail   │    │  GitHub  │    │ Fichiers │
                   │   MCP    │    │   MCP    │    │   MCP    │
                   └────┬─────┘    └────┬─────┘    └────┬─────┘
                        ↓              ↓                ↓
                     Gmail API      GitHub API    Système de fichiers
```

## Les 3 composants principaux

### 1. Le Host (application hôte)

L'application qui intègre le LLM et gère les clients MCP.

```
Exemples de hosts :
  - Claude Desktop (app Anthropic)
  - Claude.ai (interface web)
  - Cursor (éditeur de code)
  - Ton application Python/JS custom
```

Le host est responsable de :
- Gérer les connexions aux serveurs MCP
- Transmettre les capacités des serveurs au LLM
- Exécuter les appels d'outils demandés par le LLM

### 2. Le Client MCP

Le composant côté host qui maintient la connexion avec un serveur MCP.

```
1 client MCP = 1 connexion vers 1 serveur MCP
Le host peut avoir plusieurs clients MCP (un par serveur)
```

Le client :
- Établit la connexion (stdio ou HTTP/SSE)
- Découvre les capacités du serveur (outils, ressources, prompts)
- Transmet les requêtes du LLM au serveur
- Retourne les résultats au LLM

### 3. Le Serveur MCP

Le composant qui expose des capacités à n'importe quel client MCP compatible.

```
Un serveur MCP expose :
  - Des Tools    : fonctions que le LLM peut appeler
  - Des Resources: données que le LLM peut lire
  - Des Prompts  : templates de prompts réutilisables
```

## Les modes de transport

MCP supporte deux modes de communication :

### stdio (local)
```
Host → [stdin/stdout] → Serveur MCP (processus local)
```
- Serveur tourne en local sur la machine
- Communication via entrée/sortie standard
- Idéal pour les outils locaux (fichiers, bases de données locales)

### HTTP + SSE (distant)
```
Host → [HTTP/SSE] → Serveur MCP (distant, cloud)
```
- Serveur hébergé sur le cloud
- Communication via HTTP et Server-Sent Events
- Idéal pour les APIs externes (Gmail, GitHub, Slack...)

## Le protocole de communication

MCP utilise **JSON-RPC 2.0** — un format standardisé de messages JSON.

```json
// Requête du client vers le serveur
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "send_email",
    "arguments": {
      "to": "alice@example.com",
      "subject": "Bonjour",
      "body": "Comment vas-tu ?"
    }
  }
}

// Réponse du serveur
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {"type": "text", "text": "Email envoyé avec succès (ID: msg_123)"}
    ]
  }
}
```

## Cycle de vie d'une connexion MCP

```
1. INITIALISATION
   Client → "initialize" (version MCP, capacités client)
   Serveur → "initialize result" (capacités serveur)

2. DÉCOUVERTE
   Client → "tools/list"
   Serveur → liste de tous les outils disponibles

3. UTILISATION (boucle)
   LLM décide d'appeler un outil
   Client → "tools/call" (nom outil + paramètres)
   Serveur → exécute + retourne le résultat

4. FERMETURE
   Client → ferme la connexion proprement
```

> [!info] MCP est language-agnostic
> Des SDK officiels existent pour Python et TypeScript. Des SDK communautaires couvrent Go, Java, Rust, C#, Ruby...

> [!tip] Un serveur MCP = une API réutilisable
> Une fois ton serveur MCP écrit, il fonctionne avec Claude, GPT-4, Gemini, et n'importe quel LLM qui supporte MCP. Tu écris une fois, tu connectes partout.
