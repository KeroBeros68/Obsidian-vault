#ia #llmops #optimisation #coûts #latence #performance

## Optimisation des coûts et de la latence

À grande échelle, chaque token compte. Les techniques d'optimisation peuvent réduire les coûts de 50 à 90% sans dégrader la qualité.

## Comprendre les coûts LLM

```
Coût total = (tokens_input × prix_input) + (tokens_output × prix_output)

Exemples de prix (ordre de grandeur, vérifier les tarifs actuels) :
  Claude Sonnet  : ~$3/M tokens input, ~$15/M tokens output
  Claude Haiku   : ~$0.25/M tokens input, ~$1.25/M tokens output
  GPT-4o         : ~$2.5/M tokens input, ~$10/M tokens output
  GPT-4o-mini    : ~$0.15/M tokens input, ~$0.60/M tokens output

Pour 1000 requêtes avec 1000 tokens input + 500 tokens output :
  Claude Sonnet  : 1000 × (1000×$0.003 + 500×$0.015) = ~$10.5
  Claude Haiku   : 1000 × (1000×$0.00025 + 500×$0.00125) = ~$0.875
  → Haiku est 12× moins cher pour le même volume
```

## Stratégie 1 — Choisir le bon modèle

Le levier le plus puissant. Pas toutes les tâches ne nécessitent le modèle le plus puissant.

```
Tâche                          Modèle recommandé
─────────────────────────────────────────────────
Analyse complexe, raisonnement → Claude Opus / GPT-4o
Tâches standard, RAG, chatbot  → Claude Sonnet / GPT-4o-mini
Classification, extraction     → Claude Haiku / GPT-4o-mini
Résumé simple, formatage       → Claude Haiku / Mistral Small

Routage intelligent :
  if complexité(question) == "haute":
      modèle = "claude-opus"
  elif complexité(question) == "moyenne":
      modèle = "claude-sonnet"
  else:
      modèle = "claude-haiku"
```

## Stratégie 2 — Caching des réponses

Éviter de re-générer des réponses identiques ou très similaires.

### Cache exact (pour les requêtes répétées à l'identique)

```python
import hashlib
import redis

cache = redis.Redis()

def llm_avec_cache(prompt: str, ttl_secondes: int = 3600) -> str:
    # Clé de cache basée sur le hash du prompt
    clé = hashlib.md5(prompt.encode()).hexdigest()
    
    # Chercher en cache
    réponse_cachée = cache.get(clé)
    if réponse_cachée:
        return réponse_cachée.decode()
    
    # Appeler le LLM si pas en cache
    réponse = llm.invoke(prompt)
    
    # Stocker en cache
    cache.setex(clé, ttl_secondes, réponse)
    
    return réponse
```

### Prompt caching (fonctionnalité native Anthropic)

Claude supporte le **prompt caching** natif : les tokens du system prompt sont mis en cache côté Anthropic, réduisant les coûts de 90% sur la partie cachée.

```python
# Avec prompt caching Anthropic
réponse = client.messages.create(
    model="claude-sonnet-4-20250514",
    system=[
        {
            "type": "text",
            "text": system_prompt_long,          # 10 000 tokens
            "cache_control": {"type": "ephemeral"} # ← mis en cache
        }
    ],
    messages=[{"role": "user", "content": question}]
)
# Les 10 000 tokens du system prompt ne sont facturés que 10% après le premier appel
```

### Cache sémantique (pour les questions similaires)

```python
# Si deux questions ont une similarité > 0.95 → retourner la réponse cachée
def cache_sémantique(question: str) -> str | None:
    vecteur_question = embedding_model.encode(question)
    
    # Chercher dans le cache vectoriel
    résultats = cache_vectoriel.search(vecteur_question, top_k=1)
    
    if résultats and résultats[0].score > 0.95:
        return résultats[0].réponse_cachée
    
    return None
```

## Stratégie 3 — Optimiser les prompts

```
Réduire le nombre de tokens sans dégrader la qualité.

❌ Verbeux :
"Pourriez-vous s'il vous plaît analyser attentivement le texte
suivant et me fournir un résumé complet et détaillé en français ?"

✅ Concis (même résultat) :
"Résume ce texte en français :"

Économie : ~15 tokens, multiplié par 100k requêtes = 1.5M tokens économisés
```

## Stratégie 4 — Streaming

Afficher la réponse au fur et à mesure de sa génération. Ne réduit pas le coût mais améliore fortement la latence perçue.

```python
# Sans streaming → l'utilisateur attend 5 secondes, puis voit tout
réponse = llm.invoke(prompt)
afficher(réponse)

# Avec streaming → l'utilisateur voit les mots apparaître immédiatement
with llm.stream(prompt) as stream:
    for chunk in stream:
        afficher(chunk)  # affichage progressif
```

## Stratégie 5 — Traitement batch

Pour les tâches non temps réel, traiter en lot réduit les coûts.

```python
# API Batch Anthropic (50% moins cher que les appels synchrones)
batch = client.beta.messages.batches.create(
    requests=[
        {"custom_id": f"req_{i}", "params": {"model": "claude-haiku-4-5", "messages": [...]}}
        for i in range(1000)
    ]
)
# Résultats disponibles dans quelques heures → 50% de réduction
```

## Stratégie 6 — Réduire les appels LLM dans les agents

```
❌ Agent qui fait 10 appels LLM pour une tâche simple
✅ Fusionner les étapes quand possible

Avant : Étape 1 (LLM) → Étape 2 (LLM) → Étape 3 (LLM)
Après : Étape 1+2+3 fusionnées (1 LLM) → résultat direct
→ 3× moins d'appels, 3× moins de coût, 3× moins de latence
```

## Tableau de bord d'optimisation

| Technique | Réduction coût | Impact latence | Complexité |
|---|---|---|---|
| Modèle moins puissant | 80-95% | Améliore | Faible |
| Prompt caching | 50-90% sur system prompt | Améliore | Faible |
| Cache exact | 100% sur hits | Améliore | Faible |
| Cache sémantique | 30-70% selon usage | Améliore | Moyenne |
| Batch processing | 50% | Dégrade | Faible |
| Streaming | 0% | Améliore perçue | Faible |
| Optimiser prompts | 10-30% | Améliore | Moyenne |

> [!tip] Ordre d'implémentation
> 1. Choisir le bon modèle par tâche (impact maximal, effort minimal)
> 2. Activer le prompt caching si system prompt long
> 3. Ajouter un cache exact sur les requêtes fréquentes
> 4. Optimiser les prompts verbeux
> 5. Cache sémantique si le volume justifie la complexité
