#ia #agents #multi-agents #avancé

## Multi-agents

Un système multi-agents est un ensemble d'agents IA qui **collaborent**, chacun avec un rôle spécialisé, pour accomplir un objectif commun.

## Pourquoi plusieurs agents ?

```
Agent unique sur une tâche complexe :
  → Context window saturée
  → Un seul LLM ne peut pas être expert en tout
  → Pas de parallélisme
  → Erreur à une étape = tout recommencer

Système multi-agents :
  → Chaque agent a un contexte ciblé et léger
  → Chaque agent est spécialisé dans son domaine
  → Certains agents travaillent en parallèle
  → Les erreurs sont isolées par agent
```

## Les architectures multi-agents

### Architecture 1 — Hiérarchique (Orchestrateur + Sous-agents)

```
[Agent Orchestrateur]
  "Prépare une analyse de marché complète"
        ↓
   ┌────┴─────┬──────────┐
   ↓          ↓          ↓
[Agent     [Agent     [Agent
Recherche] Analyse]   Rédaction]
   ↓          ↓          ↓
   └────┬─────┴──────────┘
        ↓
[Orchestrateur synthétise]
        ↓
  Résultat final
```

**Avantage** : coordination claire, l'orchestrateur garde le contrôle.

### Architecture 2 — Pipeline séquentiel

```
[Agent 1 Collecte] → Données brutes
    ↓
[Agent 2 Nettoyage] → Données propres
    ↓
[Agent 3 Analyse] → Insights
    ↓
[Agent 4 Rapport] → Document final
```

**Avantage** : simple, chaque agent reçoit exactement ce dont il a besoin.

### Architecture 3 — Pair-à-pair (collaboration)

```
[Agent A] ←──────────────→ [Agent B]
"Expert technique"         "Expert business"
         ↓ débattent et itèrent
    [Consensus / Décision]
```

Utilisé par AutoGen pour la génération de code (un agent code, un autre relit et critique).

### Architecture 4 — Réseau d'agents spécialisés

```
[Router Agent]
    ↓ analyse la demande et route
    ├──→ [Agent Juridique]    si question légale
    ├──→ [Agent Financier]    si question finance
    ├──→ [Agent Technique]    si question tech
    └──→ [Agent Commercial]   si question vente
```

## Exemple concret : pipeline de création de contenu

```
[Brief] → Agent Recherche → collecte infos sur le sujet
              ↓
          Agent SEO → identifie les mots-clés cibles
              ↓
          Agent Rédaction → rédige l'article
              ↓
          Agent Éditorial → vérifie le style et la cohérence
              ↓
          Agent Formatage → met en forme pour WordPress
              ↓
          [Article prêt à publier]
```

## Communication entre agents

Les agents se passent des informations via :

| Mécanisme | Description |
|---|---|
| **Message passing** | Un agent envoie un message structuré à un autre |
| **État partagé** | Tous les agents lisent/écrivent dans un état commun |
| **File de tâches** | Les agents prennent des tâches dans une queue |
| **Mémoire partagée** | Vector DB commune accessible à tous les agents |

## Frameworks multi-agents

| Framework | Architecture | Points forts |
|---|---|---|
| **AutoGen** (Microsoft) | Pair-à-pair, conversations | Excellent pour code et debug |
| **CrewAI** | Hiérarchique avec rôles | Simple, intuitif, bon pour débuter |
| **LangGraph** | Graphe d'états | Très flexible, contrôle total |
| **AgentScope** (Alibaba) | Pipeline distribué | Tolérance aux pannes |

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Parallélisme → plus rapide | Complexité de coordination |
| Spécialisation → meilleure qualité | Coût élevé en tokens |
| Modularité → facile à étendre | Débogage très difficile |
| Contextes plus légers par agent | Risques de boucles et conflits |

> [!warning] Ne pas sur-architecturer
> Commence toujours par un seul agent. Passe au multi-agents uniquement si tu rencontres des limites claires : context window insuffisante, besoin de parallélisme, ou domaines trop hétérogènes pour un seul agent.

> [!tip] CrewAI pour débuter
> CrewAI est le framework le plus accessible pour découvrir les multi-agents. Tu définis des agents avec des rôles en langage naturel, et il gère la coordination automatiquement.
