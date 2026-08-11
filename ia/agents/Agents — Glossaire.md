#ia #agents #glossaire #référence

| Terme | Définition |
|---|---|
| **Agent IA** | Système basé sur un LLM qui peut planifier, utiliser des outils, mémoriser et s'auto-corriger pour atteindre un objectif de manière autonome. |
| **Autonomie** | Capacité de l'agent à décider seul de ses actions sans intervention humaine à chaque étape. |
| **Boucle agent** | Le cycle fondamental : Observation → Réflexion → Action → Observation... jusqu'à atteindre l'objectif. |
| **ReAct** | Pattern Reasoning + Acting. L'agent alterne entre réflexion (Thought) et action (Action), observe le résultat, recommence. Introduit par Yao et al. (2022). |
| **Tool / Outil** | Fonction que l'agent peut appeler pour interagir avec le monde extérieur (recherche web, API, fichiers, code...). |
| **Tool calling** | Mécanisme par lequel le LLM génère une instruction structurée pour appeler un outil avec les bons paramètres. |
| **Orchestrateur** | Composant qui gère la boucle d'exécution de l'agent : envoie le contexte au LLM, exécute les outils, boucle jusqu'à la fin. |
| **System prompt** | Instructions de base données à l'agent qui définissent son rôle, ses règles, ses outils disponibles et son format de sortie. |
| **Guardrail** | Contrainte de sécurité sur l'agent : nombre max d'itérations, timeout, actions nécessitant validation humaine. |
| **Human-in-the-loop** | Pattern où l'agent s'arrête à certaines étapes pour demander validation ou correction à un humain. |
| **Plan and Execute** | Pattern où l'agent planifie toutes les étapes d'abord, puis les exécute. Opposé au ReAct qui alterne. |
| **Reflexion** | Pattern où l'agent génère une réponse puis la critique lui-même (ou via un second agent) pour l'améliorer. |
| **Multi-agents** | Système composé de plusieurs agents qui collaborent, chacun spécialisé dans un domaine ou une tâche. |
| **Orchestrateur (multi-agents)** | Agent principal qui coordonne et délègue les tâches aux autres agents dans une architecture hiérarchique. |
| **Pattern Superviseur** | Architecture multi-agents où un orchestrateur central décide seul de la suite ; les agents spécialisés ne se parlent jamais entre eux, tous reviennent au superviseur. La plus lisible et la plus répandue. |
| **Pattern Swarm (essaim)** | Architecture multi-agents décentralisée : chaque agent peut passer la main directement à un autre sans passer par un point central — plus souple pour un parcours imprévisible, mais la coordination se disperse dans tous les agents. |
| **Pattern Hiérarchique (multi-agents)** | Architecture où des superviseurs sont imbriqués : un superviseur de haut niveau orchestre des équipes, chacune étant elle-même un graphe avec son propre superviseur — réponse au passage à l'échelle. |
| **Boucle de correction (multi-agents)** | Le superviseur renvoie le travail à un agent en amont tant qu'un critère n'est pas satisfait (ex. un verdict de fact-checking), jusqu'à un plafond de révisions qui borne la boucle. |
| **Subagent** | Agent spécialisé qui reçoit des instructions d'un orchestrateur et retourne ses résultats. |
| **AgentExecutor** | Classe LangChain qui gère la boucle d'exécution d'un agent : LLM → outil → LLM → ... |
| **LangGraph** | Extension de LangChain qui modélise l'agent comme un graphe d'états, permettant des workflows complexes avec branchements. |
| **State Graph** | Dans LangGraph, graphe dont les nœuds sont des fonctions Python qui modifient un état partagé. |
| **Crew** | Dans CrewAI, l'équipe d'agents avec leurs tâches et leur mode de collaboration (séquentiel ou hiérarchique). |
| **GroupChat** | Dans AutoGen, mécanisme permettant à plusieurs agents de participer à une conversation gérée par un manager. |
| **Context window** | Limite de tokens que le LLM peut traiter en une fois. Contrainte majeure pour les agents sur des tâches longues. |
| **Token budget** | Nombre maximum de tokens alloué à une session d'agent. Guardrail important pour contrôler les coûts. |
| **Agentic RAG** | RAG où un agent décide de manière autonome combien de fois et comment chercher dans les documents. Voir [[RAG 06 — Agentic RAG]]. |
| **Mem0** | Bibliothèque dédiée à la mémoire long terme d'un agent : extrait les faits d'une conversation, les vectorise et les range dans une base vectorielle — une brique à brancher sur n'importe quel framework d'agent. |
| **Letta (MemGPT)** | Framework d'agents construit autour de la mémoire, avec une gestion hiérarchique inspirée d'un système d'exploitation — l'agent décide lui-même ce qu'il garde en contexte et ce qu'il archive. |
| **`user_id` (mémoire)** | Clé de filtrage systématique des opérations `add`/`search` d'une mémoire long terme, garantissant que la mémoire d'un utilisateur ne se mélange jamais à celle d'un autre. |
| **Trace** | Enregistrement complet de toutes les étapes d'exécution d'un agent : prompts, outils appelés, résultats, latences. |
| **LangSmith** | Outil de traçabilité et débogage de LangChain pour observer et analyser l'exécution des agents. |
