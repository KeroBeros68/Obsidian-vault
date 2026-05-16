#ia #llmops #cycle-de-vie #workflow #bases

## Cycle de vie d'une application LLM

Chaque application LLM passe par les mêmes grandes phases, du prototype à la production stable.

## Vue d'ensemble

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  DESIGN  │──>│  BUILD   │──>│  ÉVALUA- │──>│  DEPLOY  │──>│ MONITOR  │
│          │   │          │   │   TION   │   │          │   │          │
│ Définir  │   │ Prompt   │   │ Evals    │   │ API      │   │ Traces   │
│ le cas   │   │ RAG      │   │ Tests    │   │ Guardrails│  │ Alertes  │
│ d'usage  │   │ Agents   │   │ Benchmarks│  │ Cache    │   │ Dérives  │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └────┬─────┘
     ↑                                                           │
     └─────────────── Itération continue ←───────────────────────┘
```

## Phase 1 — Design

Définir précisément ce que l'application doit faire avant d'écrire une seule ligne de code.

```
Questions à répondre :
  - Quel est le cas d'usage exact ? (support client, résumé, génération...)
  - Qui sont les utilisateurs ? (techniques, grand public, experts...)
  - Quelles sont les contraintes ? (latence max, coût max, langue...)
  - Comment mesurer le succès ? (métriques business + qualité LLM)
  - Quels sont les risques ? (hallucination, abus, données sensibles...)
```

> [!tip] La définition des métriques de succès en phase Design est le travail le plus important. Si tu ne sais pas comment mesurer, tu ne sauras pas si l'app fonctionne.

## Phase 2 — Build

Développer le prototype et itérer rapidement.

```
Étapes typiques :
  1. Choisir le modèle de base (Claude, GPT-4, Mistral...)
  2. Écrire le system prompt initial
  3. Tester manuellement avec des cas représentatifs
  4. Ajouter RAG si nécessaire
  5. Ajouter des agents si nécessaire
  6. Itérer sur le prompt jusqu'à satisfaction manuelle
```

Outils de développement :
- **LangChain / LlamaIndex** : framework applicatif
- **LangSmith** : débogage des traces
- **Promptfoo** : tests de prompts en local

## Phase 3 — Évaluation

Automatiser la mesure de la qualité avant chaque déploiement.

```
Créer un dataset d'évaluation :
  - 50 à 500 paires (entrée, sortie attendue)
  - Couvrir les cas nominaux ET les cas limites
  - Inclure des cas adversariaux (tentatives d'abus)

Définir les métriques :
  - Exactitude sur les questions factuelles
  - Fidélité aux documents sources (pour RAG)
  - Respect du format de sortie
  - Taux de refus appropriés
  - Score LLM-as-judge pour la qualité globale
```

> [!warning] Sans evals automatisés, chaque modification du prompt est un saut dans l'inconnu.

## Phase 4 — Déploiement

Mettre l'application en production de manière contrôlée.

```
Checklist avant déploiement :
  ✅ Evals passent au-dessus du seuil défini
  ✅ Guardrails input/output en place
  ✅ Rate limiting configuré
  ✅ Logging et tracing activés
  ✅ Alertes configurées
  ✅ Fallback défini (si LLM indisponible)
  ✅ Coût estimé pour le volume prévu
```

Stratégies de déploiement progressif :
```
Canary : 5% du trafic → nouveau prompt → si OK → 100%
A/B test : 50/50 entre deux versions → mesure → choisit le meilleur
Shadow : nouvelle version en parallèle sans impacter l'utilisateur
```

## Phase 5 — Monitoring

Observer le comportement en production et détecter les problèmes.

```
Ce qu'on surveille :
  - Latence p50, p95, p99
  - Taux d'erreur (erreurs API, timeouts)
  - Coût par requête et coût total
  - Qualité des réponses (evals en production)
  - Comportements inattendus (refus excessifs, hallucinations)
  - Dérive du modèle (les fournisseurs mettent à jour silencieusement)
```

## La boucle d'amélioration continue

```
Production → Logs → Analyse → Amélioration → Evals → Déploiement
    ↑                                                        │
    └────────────────────────────────────────────────────────┘
```

Sources d'amélioration :
- Feedback utilisateurs (thumbs up/down)
- Cas échoués identifiés dans les logs
- Nouvelles versions de modèles disponibles
- Nouveaux cas d'usage détectés

> [!info] Le prompt évolue toujours
> En production, un prompt n'est jamais "terminé". Les comportements inattendus des utilisateurs révèlent toujours des cas non couverts. Plan for iteration.
