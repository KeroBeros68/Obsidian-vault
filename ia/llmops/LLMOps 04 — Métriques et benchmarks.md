#ia #llmops #métriques #benchmarks #qualité

## Métriques et benchmarks

Les métriques quantifient la qualité d'une application LLM. Les bonnes métriques dépendent du cas d'usage.

## Métriques techniques (toujours mesurer)

### Latence
```
p50 (médiane)   : 50% des requêtes sont sous ce temps
p95             : 95% des requêtes sont sous ce temps → le "vrai" ressenti utilisateur
p99             : les cas les plus lents, à surveiller

Cibles typiques :
  Chatbot temps réel     : p95 < 3 secondes
  Génération de document : p95 < 30 secondes
  Batch (arrière-plan)   : pas de contrainte forte
```

### Coût par requête
```
Coût = (tokens_input × prix_input) + (tokens_output × prix_output)

Surveiller :
  - Coût moyen par requête
  - Coût total journalier / mensuel
  - Dérive du coût (si les réponses s'allongent avec le temps)
```

### Taux d'erreur
```
Erreurs API      : timeouts, rate limits, erreurs serveur
Erreurs parsing  : sortie non conforme au format attendu
Erreurs métier   : réponse hors sujet, refus non justifié
```

## Métriques de qualité RAG

Spécifiques aux applications RAG. Mesurées avec **Ragas** principalement.

| Métrique | Définition | Outil |
|---|---|---|
| **Faithfulness** | La réponse est-elle fidèle aux documents sources ? (pas d'invention) | Ragas |
| **Answer Relevance** | La réponse répond-elle vraiment à la question ? | Ragas |
| **Context Precision** | Les chunks récupérés sont-ils pertinents ? | Ragas |
| **Context Recall** | Tous les chunks nécessaires ont-ils été récupérés ? | Ragas |

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision

dataset = [
    {
        "question": "Quelle est la politique de retour ?",
        "answer": réponse_générée,
        "contexts": chunks_récupérés,
        "ground_truth": réponse_attendue
    }
]

scores = evaluate(dataset, metrics=[faithfulness, answer_relevancy, context_precision])
# → {"faithfulness": 0.92, "answer_relevancy": 0.87, "context_precision": 0.79}
```

## Métriques de qualité générale

### BLEU / ROUGE (pour la génération de texte)
```
BLEU  : mesure le chevauchement de n-grams entre la sortie et la référence
ROUGE : mesure le rappel des n-grams de la référence dans la sortie

Usage : traduction, résumé, tâches avec référence exacte
Limite : ne mesure pas la qualité sémantique, juste la similarité de surface
```

### BERTScore (similarité sémantique)
```
Mesure la similarité sémantique via des embeddings BERT
Plus fiable que BLEU/ROUGE pour évaluer le sens

from bert_score import score
P, R, F1 = score(réponses_générées, références, lang="fr")
```

### LLM-as-Judge custom (le plus flexible)

Définir ses propres critères selon le cas d'usage.

```
Pour un chatbot support client :
  - Exactitude : la réponse est-elle factuellement correcte ? (0-10)
  - Ton : le ton est-il approprié et professionnel ? (0-10)
  - Complétude : tous les aspects de la question sont-ils traités ? (0-10)
  - Sécurité : la réponse respecte-t-elle les limites définies ? (0/1)

Score global = moyenne pondérée des critères
```

## Métriques business (les plus importantes)

```
Au final, ce qui compte pour l'organisation :

  Taux de résolution    : % de questions résolues sans escalade humaine
  Satisfaction (CSAT)   : note donnée par l'utilisateur (1-5)
  Taux de réutilisation : % d'utilisateurs qui reviennent
  Temps économisé       : heures humaines remplacées par l'IA
  Coût par résolution   : coût IA vs coût humain équivalent
```

> [!tip] Pyramide des métriques
> Les métriques techniques (latence, erreurs) détectent les problèmes. Les métriques de qualité mesurent la performance LLM. Les métriques business mesurent la valeur créée. Les trois niveaux sont nécessaires.

> [!warning] Méfiance envers les métriques uniques
> Un seul chiffre (ex : "score de qualité global = 0.85") cache toujours des problèmes spécifiques. Toujours regarder les métriques par dimension et par segment de cas d'usage.
