#ia #agents #architecture #bases

## Architecture d'un agent

Tout agent IA, quel que soit le framework utilisé, repose sur les mêmes composants fondamentaux.

## Vue d'ensemble

```
┌─────────────────────────────────────────────┐
│                   AGENT                      │
│                                              │
│  ┌──────────┐    ┌──────────┐    ┌────────┐ │
│  │  Mémoire │    │   LLM    │    │ Outils │ │
│  │ (état)   │◄──►│ (cerveau)│◄──►│       │ │
│  └──────────┘    └────┬─────┘    └────────┘ │
│                       │                      │
│               ┌───────┴────────┐             │
│               │  Orchestrateur │             │
│               │  (boucle)      │             │
│               └───────┬────────┘             │
└───────────────────────┼─────────────────────┘
                        ↕
              Environnement extérieur
         (web, fichiers, APIs, humain...)
```

## Les 5 composants

### 1. Le LLM — le cerveau

C'est le moteur de raisonnement. Il reçoit le contexte complet (objectif + mémoire + résultats précédents) et décide de la prochaine action.

```
Input du LLM à chaque étape :
  - System prompt (rôle, règles, format de sortie)
  - Objectif initial
  - Historique des actions et résultats
  - Outils disponibles et leur description
  - Contexte mémoire pertinent
```

> [!info] Choix du LLM
> Les agents nécessitent des LLM puissants pour bien raisonner et utiliser les outils correctement. GPT-4o, Claude Opus/Sonnet, Gemini Pro sont les plus utilisés.

### 2. Les Outils — les mains

Les fonctions que l'agent peut appeler pour agir dans le monde réel. Chaque outil a une **description** que le LLM lit pour savoir quand et comment l'utiliser.

```python
# Exemple de définition d'outil
{
  "name": "recherche_web",
  "description": "Cherche des informations récentes sur Internet. Utilise cet outil quand tu as besoin d'informations actuelles ou que tu ne connais pas la réponse.",
  "parameters": {
    "query": "La requête de recherche"
  }
}
```

### 3. La Mémoire — le contexte

Ce que l'agent "sait" et "se souvient" à chaque instant.

| Type | Contenu | Durée |
|---|---|---|
| In-context | La conversation en cours | Session |
| Externe courte | Résumés des étapes précédentes | Session |
| Externe longue | Profil, préférences, historique | Persistant |
| Sémantique | Base de connaissances (RAG) | Persistant |

### 4. L'Orchestrateur — la boucle

Le chef d'orchestre qui gère l'exécution de la boucle agent.

```
BOUCLE PRINCIPALE :
  1. Envoyer le contexte au LLM
  2. Recevoir la décision (action ou réponse finale)
  3. Si action → exécuter l'outil → ajouter le résultat au contexte
  4. Si réponse finale → sortir de la boucle
  5. Retourner à l'étape 1
```

### 5. Les Guardrails — les limites

Les contraintes de sécurité qui encadrent l'autonomie de l'agent.

```
Exemples de guardrails :
  - Nombre maximum d'itérations (évite les boucles infinies)
  - Budget maximum de tokens
  - Actions nécessitant une validation humaine
  - Liste blanche des outils autorisés
  - Timeout par action
```

> [!warning] Les guardrails sont obligatoires en production
> Sans limites, un agent peut boucler indéfiniment, consommer des ressources excessives, ou exécuter des actions irréversibles non souhaitées.

## Le system prompt d'un agent

C'est la pièce la plus importante. Il définit le comportement de l'agent.

```
Tu es un assistant commercial expert.
Ton objectif : répondre aux emails clients et mettre à jour le CRM.

Outils disponibles :
  - lire_email(id) : lire un email
  - repondre_email(id, message) : envoyer une réponse
  - mettre_a_jour_crm(client_id, données) : mettre à jour une fiche client
  - rechercher_client(email) : trouver un client dans le CRM

Règles :
  - Ne jamais promettre un remboursement > 100€ sans validation humaine
  - Toujours chercher le client dans le CRM avant de répondre
  - Escalader à un humain si le client exprime une insatisfaction forte

Format de réponse : JSON avec les champs "action", "outil", "paramètres"
```

> [!tip] La qualité du system prompt détermine 80% de la qualité de l'agent
> Un agent mal défini dans son system prompt sera imprévisible. Soigne le rôle, les règles et surtout le format de sortie attendu.
