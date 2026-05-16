#ia #llmops #observabilité #tracing #monitoring #pratique

## Observabilité et tracing

Voir exactement ce qui se passe à l'intérieur de chaque appel LLM en production. Sans observabilité, déboguer une app LLM est quasi-impossible.

## Pourquoi c'est différent d'une app classique

```
App classique :
  Erreur → stacktrace → ligne de code → correction

App LLM :
  Mauvaise réponse → "Pourquoi ?"
    → Quel prompt a été envoyé exactement ?
    → Quels chunks RAG ont été récupérés ?
    → Quel modèle a été utilisé ?
    → Combien de tokens consommés ?
    → Quelle étape de l'agent a échoué ?
    → Sans traces : IMPOSSIBLE à répondre
```

## Les 3 niveaux d'observabilité

### Niveau 1 — Logs basiques

Le minimum vital. Chaque appel LLM est loggé.

```python
import logging
import time
import uuid

logger = logging.getLogger("llm_app")

def appel_llm_avec_log(prompt: str, user_id: str) -> str:
    trace_id = str(uuid.uuid4())
    start = time.time()
    
    logger.info({
        "trace_id": trace_id,
        "event": "llm_call_start",
        "user_id": user_id,
        "prompt_length": len(prompt),
        "model": "claude-sonnet-4-20250514"
    })
    
    try:
        réponse = llm.invoke(prompt)
        latence = time.time() - start
        
        logger.info({
            "trace_id": trace_id,
            "event": "llm_call_success",
            "latence_ms": int(latence * 1000),
            "tokens_input": réponse.usage.input_tokens,
            "tokens_output": réponse.usage.output_tokens,
            "coût_estimé": calculer_coût(réponse.usage)
        })
        
        return réponse.content
        
    except Exception as e:
        logger.error({
            "trace_id": trace_id,
            "event": "llm_call_error",
            "error": str(e)
        })
        raise
```

### Niveau 2 — Traces distribuées

Suivre toute la chaîne d'exécution d'un agent ou d'un pipeline RAG.

```
Requête utilisateur (trace_id: abc-123)
  ├── [Étape 1] Query rewriting           (span: 120ms, tokens: 150)
  ├── [Étape 2] Recherche vectorielle      (span: 45ms, résultats: 5 chunks)
  ├── [Étape 3] Re-ranking                 (span: 80ms)
  └── [Étape 4] Génération réponse         (span: 2300ms, tokens: 450)
  Total : 2545ms, 600 tokens, coût: $0.0018
```

### Niveau 3 — Évaluation en production (online evals)

Évaluer automatiquement la qualité de chaque réponse en temps réel.

```python
# Après chaque réponse LLM en production
def évaluer_réponse_production(question, réponse, chunks_rag):
    score = llm_judge(
        question=question,
        réponse=réponse,
        critères="Exactitude, pertinence, ton"
    )
    
    logger.info({
        "event": "online_eval",
        "score_qualité": score["score"],
        "faithfulness": vérifier_faithfulness(réponse, chunks_rag),
        "contient_données_sensibles": détecter_pii(réponse)
    })
```

## Outils d'observabilité

| Outil | Type | Points forts |
|---|---|---|
| **LangSmith** | SaaS | Intégration LangChain parfaite, evals intégrés |
| **Langfuse** | Open-source / SaaS | Complet, prompt management + traces + evals |
| **Arize Phoenix** | Open-source | Spécialisé RAG et agents, déploiement local |
| **Helicone** | SaaS | Proxy simple, zéro code, multi-providers |
| **Braintrust** | SaaS | Evals + traces, très bonne UI |
| **OpenTelemetry** | Standard ouvert | Intégration dans les outils existants |

## Intégration Langfuse (recommandée)

```python
from langfuse import Langfuse
from langfuse.decorators import observe, langfuse_context

langfuse = Langfuse()

@observe()  # ← instrumente automatiquement la fonction
def pipeline_rag(question: str, user_id: str) -> str:
    
    # Ajouter du contexte à la trace
    langfuse_context.update_current_trace(
        user_id=user_id,
        tags=["rag", "support-client"]
    )
    
    @observe(name="retrieval")
    def récupérer_chunks(q):
        return vector_db.search(q, top_k=5)
    
    @observe(name="generation")
    def générer_réponse(q, chunks):
        return llm.invoke(f"Contexte: {chunks}\n\nQuestion: {q}")
    
    chunks = récupérer_chunks(question)
    réponse = générer_réponse(question, chunks)
    
    # Score de satisfaction (si feedback utilisateur disponible)
    langfuse_context.score_current_trace(
        name="user_feedback",
        value=None  # rempli plus tard via le feedback UI
    )
    
    return réponse
```

## Dashboard de monitoring

Ce qu'il faut surveiller en temps réel.

```
┌─────────────────────────────────────────────────────┐
│                 DASHBOARD LLMOps                     │
│                                                      │
│  Latence p95    Taux erreur    Coût/jour             │
│  ████ 2.3s      ██ 0.8%        ███ $42               │
│                                                      │
│  Qualité moyenne    Tokens/requête    Requêtes/h      │
│  ████████ 8.2/10   ████ 820          ███ 1240        │
│                                                      │
│  Alertes actives :                                   │
│  ⚠️  p95 latence > 5s depuis 10 min (région EU)     │
└─────────────────────────────────────────────────────┘
```

## Alertes à configurer

```yaml
alertes:
  - nom: "Latence élevée"
    condition: "p95_latence > 5000ms pendant 5 minutes"
    action: "PagerDuty + Slack #ops"
    
  - nom: "Taux d'erreur élevé"
    condition: "taux_erreur > 5% sur 10 minutes"
    action: "PagerDuty"
    
  - nom: "Coût journalier anormal"
    condition: "coût_jour > coût_moyen_7j * 2"
    action: "Slack #finance"
    
  - nom: "Qualité dégradée"
    condition: "score_moyen_1h < 0.7"
    action: "Slack #produit"
    
  - nom: "Dérive du modèle"
    condition: "score_moyen_7j baisse de 10% vs 7j précédents"
    action: "Email équipe + ticket investigation"
```

> [!warning] La dérive silencieuse des modèles
> Les fournisseurs LLM (OpenAI, Anthropic) mettent à jour leurs modèles sans toujours le signaler clairement. Une surveillance continue de la qualité permet de détecter une dégradation avant que les utilisateurs ne se plaignent.
