#ia #fine-tuning #décision #comparaison #bases

## Quand fine-tuner vs autres approches

La décision de fine-tuner est l'une des plus importantes en LLMOps. Mal choisir coûte cher en temps et en argent.

## L'arbre de décision

```
Le modèle de base répond-il correctement avec un bon prompt ?
        │
        ├── OUI → Reste sur le prompting ✅
        │
        └── NON → Pourquoi ?
                    │
                    ├── Manque d'informations récentes ou internes
                    │   → Utilise le RAG ✅
                    │
                    ├── Le style/ton n'est pas le bon
                    │   → Fine-tuning ✅
                    │
                    ├── Le format de sortie est inconsistant
                    │   → Few-shot dans le prompt d'abord
                    │   → Fine-tuning si le few-shot ne suffit pas
                    │
                    ├── Le contexte est trop long pour le prompt
                    │   → RAG ou Fine-tuning selon le cas
                    │
                    └── Besoin de performances/coût inférieurs
                        → Fine-tuning sur un petit modèle ✅
```

## Comparaison détaillée

| Critère | Prompting | RAG | Fine-tuning |
|---|---|---|---|
| **Coût de mise en place** | Gratuit | Moyen | Élevé |
| **Délai** | Immédiat | Quelques jours | Jours à semaines |
| **Mise à jour** | Instantanée | Quasi-instantanée | Ré-entraînement |
| **Connaissances récentes** | ❌ | ✅ | ❌ |
| **Style/ton personnalisé** | Partiel | ❌ | ✅ |
| **Coût par requête** | Élevé (long prompt) | Moyen | Faible (prompt court) |
| **Données nécessaires** | Aucune | Documents | Dataset labellisé |
| **Comportement cohérent** | Variable | Variable | ✅ Très stable |

## Les bons cas d'usage du fine-tuning

### ✅ Cas 1 — Style et ton de marque très spécifique
```
Situation : ton assistant doit toujours répondre avec la voix
            de ta marque, un humour particulier, un style narratif unique.

Pourquoi pas le prompting ? → Le style dérive sur les longues conversations.
Pourquoi pas le RAG ?       → Le RAG ne change pas le style du LLM.
Fine-tuning : ✅ Le style est "gravé" dans les poids du modèle.
```

### ✅ Cas 2 — Format de sortie structuré et fiable
```
Situation : l'app a besoin d'un JSON strictement formaté à chaque réponse,
            avec des champs précis, sans variabilité.

Pourquoi pas le prompting ? → Inconsistant même avec des exemples.
Fine-tuning : ✅ Le modèle apprend à toujours produire le bon format.
```

### ✅ Cas 3 — Domaine très spécialisé avec jargon spécifique
```
Situation : assistant médical, juridique, ou pour un jeu vidéo spécifique
            avec un vocabulaire très spécialisé que le modèle de base
            ne maîtrise pas assez.

Fine-tuning : ✅ Permet d'injecter le jargon et les conventions du domaine.
```

### ✅ Cas 4 — Réduire les coûts à grande échelle
```
Situation : 1M de requêtes/jour avec un long system prompt (5000 tokens).
            Fine-tuner un petit modèle = même qualité, prompt minimal.

Économie possible :
  GPT-4o + prompt 5000 tokens × 1M/j = très coûteux
  GPT-4o-mini fine-tuné + prompt 50 tokens × 1M/j = 10-50× moins cher
```

### ✅ Cas 5 — Latence critique
```
Situation : l'app nécessite des réponses en < 500ms.
            Un petit modèle fine-tuné est bien plus rapide qu'un grand modèle.
```

## Les mauvais cas d'usage du fine-tuning

### ❌ Cas 1 — Injecter des faits précis
```
❌ "Je vais fine-tuner pour que le modèle connaisse notre catalogue produit"
→ Les LLM sont mauvais pour mémoriser des facts précis via le fine-tuning
→ Les faits peuvent être confondus ou mal restitués
✅ Utiliser le RAG pour les données factuelles
```

### ❌ Cas 2 — Résoudre un problème de prompt
```
❌ "Le modèle ne fait pas ce que je veux → je vais le fine-tuner"
→ D'abord investir dans le prompt engineering
→ Le fine-tuning ne compense pas un mauvais prompt
✅ Tester 20 variantes de prompts avant de considérer le fine-tuning
```

### ❌ Cas 3 — Données qui changent souvent
```
❌ "Je vais fine-tuner sur nos données clients" (mises à jour quotidiennes)
→ Chaque mise à jour = ré-entraînement coûteux
✅ RAG : mise à jour instantanée de l'index
```

> [!tip] Le test décisif
> Avant de fine-tuner, pose-toi la question : "Avec 100 exemples en few-shot dans le prompt, le résultat serait-il satisfaisant ?" Si oui → fine-tune. Si non → le problème vient ailleurs.
