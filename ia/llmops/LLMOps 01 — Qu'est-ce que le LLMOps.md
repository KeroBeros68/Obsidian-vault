#ia #llmops #bases #définition

## Qu'est-ce que le LLMOps ?

LLMOps = **Large Language Model Operations**
L'ensemble des pratiques, outils et processus pour **développer, déployer, monitorer et maintenir** des applications basées sur des LLM en production.

## Le problème qu'il résout

Mettre un LLM en production est fondamentalement différent d'un logiciel classique.

```
Logiciel classique :
  Code → Tests → Déploiement → ça marche ou ça ne marche pas
  Comportement : déterministe et prévisible
  Bug : reproductible, corrigeable par le code

Application LLM :
  Prompt → LLM → Réponse → parfois bonne, parfois non
  Comportement : probabiliste et variable
  Bug : difficile à reproduire, corrigeable par le prompt ou les données
```

LLMOps fournit les outils pour gérer cette **imprévisibilité** à grande échelle.

## LLMOps vs MLOps

| | MLOps | LLMOps |
|---|---|---|
| **Modèles** | Entraînés from scratch | Principalement des modèles fondation existants |
| **Personnalisation** | Entraînement complet | Prompting, RAG, fine-tuning |
| **Évaluation** | Métriques numériques (accuracy, F1...) | Métriques + évaluation qualitative (LLM-as-judge) |
| **Artefact principal** | Le modèle | Le prompt + les données |
| **Drift** | Dérive des données d'entrée | Dérive des modèles (mises à jour fournisseur) |
| **Coût** | Coût d'entraînement | Coût par token (inference) |

## Les 5 piliers du LLMOps

```
┌─────────────────────────────────────────────────┐
│                   LLMOps                        │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │ ÉVALUA-  │  │ PROMPT   │  │OBSERVABILITÉ │   │
│  │  TION    │  │MANAGEMENT│  │  & TRACING   │   │
│  └──────────┘  └──────────┘  └──────────────┘   │
│                                                 │
│  ┌──────────────────┐  ┌──────────────────────┐ │
│  │   OPTIMISATION   │  │   SÉCURITÉ &         │ │
│  │  COÛT / LATENCE  │  │   GUARDRAILS         │ │
│  └──────────────────┘  └──────────────────────┘ │
└─────────────────────────────────────────────────┘
```

### 1. Évaluation (Evals)
Comment mesurer si l'application répond bien ? Automatiser cette mesure à grande échelle.

### 2. Prompt Management
Versionner, tester et déployer les prompts comme du code.

### 3. Observabilité & Tracing
Voir ce qui se passe à l'intérieur de chaque appel LLM en production.

### 4. Optimisation coût/latence
Réduire les coûts en tokens et améliorer la vitesse de réponse.

### 5. Sécurité & Guardrails
Protéger l'application contre les abus, les injections de prompt et les réponses dangereuses.

## Quand le LLMOps devient nécessaire

```
Prototype (1 utilisateur, toi) → pas besoin de LLMOps
    ↓
Beta (10-100 utilisateurs) → evals basiques, quelques logs
    ↓
Production (1000+ utilisateurs) → LLMOps complet obligatoire
    ↓
Scale (100k+ utilisateurs) → optimisation coût/latence critique
```

> [!tip] Ne pas sur-ingéniérer dès le début
> Commence simple : quelques logs, un eval basique. Ajoute de la complexité LLMOps au fur et à mesure que les besoins réels émergent.

> [!info] LLMOps est un domaine jeune
> Il évolue très vite. Les outils de 2023 sont déjà dépassés par ceux de 2025. Reste à jour sur les nouvelles pratiques.
