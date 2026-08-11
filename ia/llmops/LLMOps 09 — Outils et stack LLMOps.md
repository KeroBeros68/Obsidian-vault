#ia #llmops #outils #stack #référence

## Outils et stack LLMOps

La carte complète des outils disponibles par catégorie, avec des recommandations selon le contexte.

## Vue d'ensemble de la stack

```
┌─────────────────────────────────────────────────────────────────┐
│                        STACK LLMOps                              │
│                                                                  │
│  DÉVELOPPEMENT          ÉVALUATION           PRODUCTION          │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐   │
│  │ LangChain     │    │ Promptfoo     │    │ Langfuse      │   │
│  │ LlamaIndex    │    │ DeepEval      │    │ LangSmith     │   │
│  │ LangGraph     │    │ Ragas (RAG)   │    │ Arize Phoenix │   │
│  └───────────────┘    └───────────────┘    └───────────────┘   │
│                                                                  │
│  PROMPT MGMT            GUARDRAILS          INFRASTRUCTURE       │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐   │
│  │ Langfuse      │    │ Guardrails AI │    │ AWS Bedrock   │   │
│  │ LangSmith Hub │    │ NeMo Guards   │    │ Azure OpenAI  │   │
│  │ Git + YAML    │    │ LlamaGuard    │    │ GCP Vertex AI │   │
│  └───────────────┘    └───────────────┘    └───────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Catégorie 1 — Frameworks applicatifs

| Outil | Usage | Quand choisir |
|---|---|---|
| **LangChain** | Chaînes, agents, RAG en Python/JS | Cas d'usage variés, équipe Python |
| **LlamaIndex** | RAG et pipelines de données | Focus données et RAG complexe |
| **LangGraph** | Agents avec état, workflows complexes | Contrôle fin sur les agents |
| **Haystack** | Pipelines NLP et RAG | Orienté prod, modèles open-source |
| **Semantic Kernel** | LLM en .NET et Python | Équipes Microsoft/Azure |

## Catégorie 2 — Observabilité et tracing

| Outil | Type | Points forts |
|---|---|---|
| **Langfuse** | Open-source / SaaS | Tout-en-un : traces + evals + prompts. Self-hostable. |
| **LangSmith** | SaaS | Intégration LangChain parfaite, UI excellente |
| **Arize Phoenix** | Open-source | Spécialisé RAG et agents, OTEL compatible |
| **Helicone** | SaaS | Proxy zero-code, multi-providers, analytics |
| **Braintrust** | SaaS | Evals + traces, bonne ergonomie |

## Catégorie 3 — Évaluation (Evals)

| Outil | Type | Points forts |
|---|---|---|
| **Promptfoo** | Open-source | CI/CD natif, multi-modèles, gratuit |
| **DeepEval** | Open-source | 14 métriques built-in, LLM-as-judge |
| **Ragas** | Open-source | Spécialisé RAG (faithfulness, relevance...) |
| **TruLens** | Open-source | RAG + agents, TruEra |
| **Evals** (OpenAI) | Open-source | Benchmarks variés, communauté active |

## Catégorie 4 — Prompt Management

| Outil | Points forts |
|---|---|
| **Langfuse** | Versioning + déploiement + A/B test |
| **LangSmith Hub** | Partage de prompts, intégration LangChain |
| **Git + YAML** | Simple, gratuit, suffisant pour débuter |
| **PromptLayer** | Historique, analytics, collaboration |

## Catégorie 5 — Guardrails et sécurité

| Outil | Points forts |
|---|---|
| **Guardrails AI** | Framework Python déclaratif, très flexible |
| **NeMo Guardrails** | Guardrails conversationnels, NVIDIA |
| **LlamaGuard** | Modèle Meta fine-tuné pour la sécurité |
| **Azure Content Safety** | API cloud managed, facile à intégrer |
| **Rebuff** | Détection d'injection de prompt spécifiquement |

## Catégorie 6 — Infrastructure et déploiement

| Outil | Points forts |
|---|---|
| **AWS Bedrock** | LLM managed sur AWS, sécurité enterprise |
| **Azure OpenAI** | OpenAI + compliance Microsoft, RGPD EU |
| **GCP Vertex AI** | Gemini + modèles tiers sur Google Cloud |
| **Replicate** | Déployer des modèles open-source facilement |
| **Modal** | Infra serverless Python pour LLM custom |
| **LiteLLM** | Proxy unifié pour tous les LLM (même API) |

## Stack recommandée par profil

### Débutant / Solo developer
```
Framework     : LangChain ou LlamaIndex
Observabilité : Langfuse (plan gratuit)
Evals         : Promptfoo (open-source)
Prompts       : Git + YAML
Guardrails    : Validation manuelle dans le code
```

### Startup / Petite équipe
```
Framework     : LangChain + LangGraph
Observabilité : Langfuse (self-hosted ou cloud)
Evals         : DeepEval + Ragas pour le RAG
Prompts       : Langfuse prompt management
Guardrails    : Guardrails AI
Infrastructure: AWS Bedrock ou Azure OpenAI
```

### Enterprise
```
Framework     : LangGraph ou Semantic Kernel
Observabilité : LangSmith + OpenTelemetry
Evals         : Pipeline CI/CD complet avec DeepEval
Prompts       : LangSmith Hub + processus de review
Guardrails    : NeMo Guardrails + LlamaGuard + Azure Content Safety
Infrastructure: AWS Bedrock / Azure OpenAI (compliance)
Sécurité      : Audit trail complet, RBAC, chiffrement
```

## LiteLLM — le proxy universel

LiteLLM permet d'utiliser la même interface pour tous les LLM. Très utile pour changer de fournisseur sans toucher au code applicatif. Voir [[LiteLLM — Index des fiches]] pour un module dédié (streaming, historique de conversation, gestion des erreurs).

```python
from litellm import completion

# Même code pour tous les LLM
réponse = completion(
    model="claude-sonnet-4-20250514",  # ou "gpt-4o" ou "gemini/gemini-pro"
    messages=[{"role": "user", "content": "Bonjour !"}]
)

# Changer de modèle = changer juste la chaîne "model"
# Aucun autre changement de code
```

> [!tip] Commencer avec Langfuse + Promptfoo
> Ces deux outils couvrent 80% des besoins LLMOps avec une complexité minimale. Langfuse pour observer ce qui se passe, Promptfoo pour tester avant de déployer. Les deux sont open-source et gratuits pour débuter.
