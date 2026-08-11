#ia #fine-tuning #coûts #infrastructure #pratique

## Coûts et infrastructure

Le fine-tuning a un coût non-négligeable. Le calculer avant de commencer évite les mauvaises surprises.

## Structure des coûts

```
Coût total = Coût d'entraînement + Coût de déploiement + Coût de maintenance

Coût d'entraînement  : GPU × heures × tarif
Coût de déploiement  : hébergement du modèle fine-tuné
Coût de maintenance  : ré-entraînements réguliers + monitoring
```

## Coûts API cloud (entraînement managé)

### OpenAI Fine-tuning (ordre de grandeur 2025)

```
Modèle                    Entraînement          Inférence (input/output)
─────────────────────────────────────────────────────────────────────────
gpt-4o-mini fine-tuning   ~$0.003/1k tokens     ~$0.30 / ~$1.20 per 1M
gpt-4o fine-tuning        ~$0.025/1k tokens     ~$3.75 / ~$15 per 1M

Exemple concret :
  Dataset : 500 exemples × 800 tokens = 400 000 tokens
  3 epochs → 1 200 000 tokens d'entraînement
  Coût gpt-4o-mini : 1200 × $0.003 = ~$3.60 par run
  
  → Un fine-tuning complet (3-5 runs pour tuner) ≈ $15-20
```

### Calcul du ROI

```python
def calculer_roi_fine_tuning(
    nb_requêtes_jour: int,
    tokens_input_moy: int,
    tokens_output_moy: int,
    coût_entraînement: float
):
    # Coûts modèle de base (ex: gpt-4o)
    coût_base_input  = 2.50 / 1_000_000  # $/token
    coût_base_output = 10.0 / 1_000_000
    
    # Coûts modèle fine-tuné (ex: gpt-4o-mini fine-tuné)
    coût_ft_input    = 0.30 / 1_000_000
    coût_ft_output   = 1.20 / 1_000_000
    
    coût_base_jour = nb_requêtes_jour * (
        tokens_input_moy * coût_base_input +
        tokens_output_moy * coût_base_output
    )
    
    coût_ft_jour = nb_requêtes_jour * (
        tokens_input_moy * coût_ft_input +
        tokens_output_moy * coût_ft_output
    )
    
    économie_jour = coût_base_jour - coût_ft_jour
    jours_amortissement = coût_entraînement / économie_jour
    
    return {
        "coût_base_mois": coût_base_jour * 30,
        "coût_ft_mois": coût_ft_jour * 30,
        "économie_mois": économie_jour * 30,
        "amortissement_jours": jours_amortissement
    }

# Exemple : 10k requêtes/jour, 1000 tokens input, 300 tokens output
résultat = calculer_roi_fine_tuning(10_000, 1_000, 300, 20)
# → coût_base_mois: $900, coût_ft_mois: $126, économie_mois: $774
# → amortissement en < 1 jour !
```

## Coûts open-source (GPU cloud)

### Estimation par modèle et GPU

```
GPU A100 (80GB) : ~$2-3/heure selon plateforme
GPU H100 (80GB) : ~$4-6/heure

Temps de fine-tuning estimé (1000 exemples, 3 epochs) :
  Mistral 7B  + LoRA  sur 1×A100  → ~1-2 heures   → ~$3-6
  LLaMA 13B   + LoRA  sur 2×A100  → ~2-4 heures   → ~$12-24
  LLaMA 70B   + QLoRA sur 1×A100  → ~8-16 heures  → ~$25-50
  LLaMA 70B   + LoRA  sur 4×A100  → ~4-8 heures   → ~$35-70
```

### Coûts de déploiement open-source

```
Déploiement Mistral 7B fine-tuné :
  GPU A10 (24GB)  : ~$0.50/heure × 24h × 30j = ~$360/mois
  Optimisation vLLM → throughput ×5 → 1 GPU sert 5× plus d'utilisateurs

Alternatives économiques :
  Ollama (local)    : gratuit si tu as le matériel
  Replicate         : pay-per-use, ~$0.0002-0.0005/token
  Together AI       : ~$0.0002/token pour les petits modèles
  Hugging Face TGI  : self-hosted, pay as you go
```

> [!info] Dimensionner le matériel nécessaire pour Ollama
> Voir [[Ollama 01 — Prérequis matériels]] pour les tableaux RAM/VRAM selon la taille du modèle avant d'estimer le coût réel d'un déploiement local.

## Infrastructure recommandée par cas d'usage

### Cas 1 — Prototype / Validation (< $100)
```
Entraînement : Google Colab Pro+ ou RunPod
Modèle       : Mistral 7B + QLoRA ou OpenAI gpt-4o-mini FT
Déploiement  : Replicate ou Hugging Face Spaces
Monitoring   : Langfuse (plan gratuit)
```

### Cas 2 — Production légère (< $500/mois)
```
Entraînement : Modal ou RunPod
Modèle       : LLaMA 13B + LoRA ou OpenAI gpt-4o-mini FT
Déploiement  : Together AI ou 1 GPU dédié RunPod
Monitoring   : Langfuse cloud
```

### Cas 3 — Production à échelle (> $1000/mois)
```
Entraînement : AWS SageMaker ou GCP Vertex AI
Modèle       : LLaMA 70B + LoRA ou contrat Enterprise Anthropic/OpenAI
Déploiement  : AWS Bedrock, Azure OpenAI, ou cluster GPU propre
Monitoring   : LangSmith + Datadog
```

## Deux risques souvent oubliés

Au-delà du coût GPU, deux points méritent l'attention de tout ingénieur avant de lancer un fine-tuning en production.

> [!warning] Fuite de données d'entraînement
> Un modèle fine-tuné peut restituer, mot pour mot, des exemples de son dataset d'entraînement — un risque direct si ce dataset contient des données sensibles (informations clients, secrets internes). Voir [[LLMOps 08 — Sécurité et guardrails en production]] pour les mécanismes de guardrails applicables en sortie, qui restent une protection complémentaire et non un substitut à un dataset d'entraînement nettoyé de tout contenu sensible en amont.

> [!warning] Reproductibilité : versionner dataset, seeds et hyperparamètres
> Sans traçabilité de la version exacte du dataset, du seed aléatoire et des hyperparamètres utilisés, un résultat de fine-tuning devient impossible à rejouer ou à auditer — un problème qui ne se révèle généralement qu'au moment où il faut diagnostiquer une régression ou reproduire un bon résultat obtenu plusieurs semaines plus tôt.

## Checklist avant de commencer

```
✅ J'ai calculé le ROI : l'économie justifie le coût d'entraînement
✅ J'ai au moins 100 exemples de haute qualité
✅ J'ai séparé mon dataset en train/validation
✅ J'ai défini mes métriques de succès avant de commencer
✅ J'ai une baseline claire (score du modèle de base)
✅ J'ai prévu un budget pour 3-5 runs (la première tentative réussit rarement)
✅ J'ai un plan de déploiement et de monitoring post-entraînement
✅ J'ai vérifié que le RAG ne suffirait pas (moins coûteux)
✅ J'ai versionné dataset, seeds et hyperparamètres pour chaque run
✅ J'ai vérifié l'absence de données sensibles dans le dataset d'entraînement
```

> [!tip] Budget de contingence
> Prévois toujours 2-3× le budget estimé. La première version du dataset est rarement parfaite, les hyperparamètres nécessitent des ajustements, et les résultats peuvent nécessiter plusieurs itérations.

> [!warning] Le fine-tuning n'est jamais "terminé"
> En production, le modèle fine-tuné nécessite des ré-entraînements réguliers quand les données évoluent ou que les performances dérivent. Prévois ce coût récurrent dès le départ.
