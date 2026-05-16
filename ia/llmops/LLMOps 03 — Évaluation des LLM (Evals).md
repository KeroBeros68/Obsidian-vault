#ia #llmops #evals #évaluation #qualité

## Évaluation des LLM (Evals)

Les evals sont des tests automatisés qui mesurent la qualité des réponses d'une application LLM. C'est le composant le plus important du LLMOps.

## Pourquoi les evals sont critiques

```
Sans evals :
  Modifier le prompt → "J'espère que c'est mieux..."
  Changer de modèle  → "Semble bien fonctionner..."
  Déploiement        → Basé sur l'intuition

Avec evals :
  Modifier le prompt → Score : 0.72 → 0.81 (+12%) ✅ Déployer
  Changer de modèle  → Score : 0.81 → 0.79 (-2%) ❌ Annuler
  Déploiement        → Basé sur des données
```

## Les 3 types d'evals

### Type 1 — Evals déterministes (exact match)

La sortie correcte est connue à l'avance et vérifiable exactement.

```python
# Exemples de cas déterministes
cas_test = [
    {
        "input": "Quelle est la capitale de la France ?",
        "expected": "Paris",
        "check": lambda output, expected: expected.lower() in output.lower()
    },
    {
        "input": "Formate ce JSON : {name: 'Alice', age: 30}",
        "expected": '{"name": "Alice", "age": 30}',
        "check": lambda output, expected: json.loads(output) == json.loads(expected)
    }
]
```

- ✅ Rapide, peu coûteux, 100% reproductible
- ❌ Limité aux tâches avec réponse unique correcte

### Type 2 — Evals basés sur des règles (heuristiques)

Vérifier des propriétés structurelles ou de format sans connaître la réponse exacte.

```python
def eval_format_json(output: str) -> bool:
    """La sortie est-elle un JSON valide ?"""
    try:
        json.loads(output)
        return True
    except:
        return False

def eval_longueur(output: str, min_mots: int, max_mots: int) -> bool:
    """La réponse respecte-t-elle les contraintes de longueur ?"""
    nb_mots = len(output.split())
    return min_mots <= nb_mots <= max_mots

def eval_pas_hallucination_url(output: str) -> bool:
    """La réponse ne contient-elle pas d'URLs inventées ?"""
    import re
    urls = re.findall(r'https?://\S+', output)
    for url in urls:
        # Vérifier que l'URL existe réellement
        try:
            requests.head(url, timeout=3)
        except:
            return False
    return True

def eval_langue(output: str, langue_attendue: str = "fr") -> bool:
    """La réponse est-elle dans la bonne langue ?"""
    from langdetect import detect
    return detect(output) == langue_attendue
```

### Type 3 — LLM-as-Judge (évaluation par un LLM)

Un LLM tiers évalue la qualité de la réponse selon des critères définis.

```python
def llm_judge(question: str, réponse: str, critères: str) -> dict:
    prompt = f"""Tu es un évaluateur expert. Note la réponse suivante.

Question posée : {question}

Réponse à évaluer : {réponse}

Critères d'évaluation : {critères}

Réponds UNIQUEMENT en JSON :
{{
  "score": <0 à 10>,
  "justification": "<explication courte>",
  "points_forts": ["<point 1>", "<point 2>"],
  "points_faibles": ["<point 1>"]
}}"""
    
    résultat = llm.invoke(prompt)
    return json.loads(résultat)

# Utilisation
score = llm_judge(
    question="Explique le machine learning à un débutant",
    réponse=réponse_app,
    critères="Clarté, précision, absence de jargon, exemples concrets"
)
# → {"score": 8, "justification": "Claire et bien structurée...", ...}
```

- ✅ Évalue la qualité subjective (clarté, ton, pertinence)
- ✅ Scalable sur des milliers de cas
- ❌ Plus coûteux en tokens, biais possible du modèle juge

## Le dataset d'évaluation

La fondation de tous les evals. À construire avec soin.

```
Structure minimale :
  - 50 cas pour un prototype
  - 200-500 cas pour la production
  - 1000+ cas pour les systèmes critiques

Composition idéale :
  60% cas nominaux (ce que les utilisateurs feront en général)
  20% cas limites (questions ambiguës, entrées mal formées)
  10% cas adversariaux (tentatives d'abus, injections de prompt)
  10% cas de régression (bugs passés à ne pas reproduire)
```

> [!warning] Ne jamais évaluer sur les données d'entraînement du prompt
> Si tu as conçu ton prompt en regardant les mêmes exemples que ceux de ton eval, tes scores seront biaisés. Le dataset d'eval doit être indépendant.

## Evals en CI/CD

Intégrer les evals dans le pipeline de déploiement.

```yaml
# .github/workflows/llm-eval.yml
- name: Run LLM Evals
  run: python run_evals.py --dataset eval_dataset.json --threshold 0.80
  
# Si le score moyen tombe sous 0.80 → le déploiement est bloqué
```

## Outils d'évaluation

| Outil | Points forts |
|---|---|
| **LangSmith** | Intégré LangChain, evals + traces |
| **Promptfoo** | Open-source, CI/CD natif, multi-modèles |
| **Ragas** | Spécialisé évaluation RAG (faithfulness, relevance...) |
| **DeepEval** | Framework complet, LLM-as-judge intégré |
| **TruLens** | Évaluation RAG et agents, open-source |

> [!tip] Commencer avec Promptfoo
> Promptfoo est gratuit, open-source, et permet de tester des prompts contre plusieurs modèles simultanément. Idéal pour débuter les evals.
