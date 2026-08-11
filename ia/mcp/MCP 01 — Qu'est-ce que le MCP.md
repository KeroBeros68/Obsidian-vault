#ia #mcp #bases #définition

## Qu'est-ce que le MCP ?

MCP = **Model Context Protocol**
Protocole open-source créé par Anthropic, annoncé le 25 novembre 2024, pour standardiser la façon dont les LLM se connectent aux outils et données externes.

## Le problème qu'il résout

Avant MCP, chaque intégration LLM + outil était **custom et unique**.

```
Sans MCP :
  Claude  ──[code custom A]──> Gmail
  Claude  ──[code custom B]──> Notion
  Claude  ──[code custom C]──> GitHub
  GPT-4   ──[code custom D]──> Gmail     ← tout refaire !
  Gemini  ──[code custom E]──> Gmail     ← encore !

Résultat : N modèles × M outils = N×M intégrations à maintenir
```

```
Avec MCP :
  Claude  ──[MCP]──> Serveur Gmail MCP
  GPT-4   ──[MCP]──> Serveur Gmail MCP  ← même serveur !
  Gemini  ──[MCP]──> Serveur Gmail MCP  ← même serveur !

Résultat : N modèles + M serveurs = N+M composants seulement
```

> MCP est aux LLM ce que l'USB est aux ordinateurs — une prise universelle.

## Analogie USB

```
Avant USB : chaque périphérique avait son propre connecteur
  Imprimante → port parallèle
  Souris     → port PS/2
  Clavier    → port DIN
  → Chaos, incompatibilité, dépendance au fabricant

Après USB : un seul standard pour tout
  Imprimante → USB
  Souris     → USB
  Clavier    → USB
  → Plug and play, interopérabilité totale

MCP = USB pour les LLM et leurs outils
```

## Ce que MCP permet concrètement

Un LLM connecté via MCP peut :

- Lire et écrire des fichiers sur ton ordinateur
- Accéder à des bases de données
- Appeler des APIs externes (GitHub, Slack, Gmail, Notion...)
- Exécuter des commandes système
- Naviguer sur le web
- Interagir avec n'importe quel service qui expose un serveur MCP

## Qui l'utilise ?

MCP est déjà adopté par :

| Outil | Usage |
|---|---|
| **Claude Desktop** | Se connecte à des serveurs MCP locaux |
| **Claude.ai** | Connecteurs MCP dans l'interface web |
| **Cursor** | MCP pour les outils de développement |
| **VS Code (Copilot)** | Support MCP en cours |
| **LangChain** | Intégration native MCP |
| **Zed, Cody, Continue** | Éditeurs de code avec support MCP |

## L'écosystème de serveurs MCP

Des centaines de serveurs MCP sont déjà disponibles en open-source :

```
Productivité : Gmail, Google Calendar, Notion, Slack, Linear
Développement : GitHub, GitLab, Docker, Kubernetes, Terraform
Données : PostgreSQL, SQLite, MySQL, MongoDB, Redis
Fichiers : système de fichiers local, Google Drive, S3
Web : Brave Search, Puppeteer, Playwright (navigation web)
IA : Hugging Face, Replicate
```

> [!info] Anthropic a publié MCP en open-source
> Le protocole est libre, documenté et adopté par l'industrie. N'importe qui peut créer un serveur MCP pour n'importe quel service.

> [!tip] Tu utilises déjà MCP
> Dans cette conversation avec Claude, les connecteurs Google Calendar, Gmail et Google Drive visibles dans l'interface sont des serveurs MCP.
