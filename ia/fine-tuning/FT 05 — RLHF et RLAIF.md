#ia #fine-tuning #rlhf #rlaif #alignement #avancé

## RLHF et RLAIF

Techniques d'alignement qui entraînent le modèle à produire des réponses **préférées** par les humains (ou par un autre LLM), plutôt que de simplement imiter des exemples.

## Pourquoi aller au-delà du SFT ?

```
SFT (Supervised Fine-Tuning) :
  Le modèle apprend à imiter des exemples parfaits.
  Problème : comment définir "parfait" pour des questions subjectives ?
  
  "Explique la relativité" → 100 experts donnent 100 réponses différentes
  Toutes bonnes, mais lesquelles le modèle doit-il imiter ?

RLHF :
  On ne montre plus UNE bonne réponse.
  On montre DEUX réponses et on dit laquelle est meilleure.
  Le modèle apprend à optimiser vers les préférences humaines.
```

## RLHF — Reinforcement Learning from Human Feedback

Technique utilisée par OpenAI (ChatGPT) et Anthropic (Claude) pour aligner leurs modèles.

### Les 3 étapes

#### Étape 1 — SFT initial
```
Partir d'un modèle pré-entraîné
Fine-tuner sur des démonstrations humaines de haute qualité
→ Modèle SFT : sait globalement quoi faire
```

#### Étape 2 — Entraînement du Reward Model
```
Collecter des préférences humaines :
  Question : "Explique le machine learning"
  Réponse A : explication technique et dense
  Réponse B : explication simple avec une analogie
  Annotateur : "B est meilleure"

Entraîner un modèle de récompense (Reward Model) :
  Input  : (question, réponse)
  Output : score de qualité numérique

Le Reward Model apprend à prédire les préférences humaines.
```

#### Étape 3 — Optimisation par RL (PPO)
```
Utiliser le Reward Model comme signal de récompense :

  Modèle génère une réponse
        ↓
  Reward Model l'évalue → score
        ↓
  PPO (Proximal Policy Optimization) met à jour le modèle
  pour maximiser le score
        ↓
  Le modèle apprend à générer des réponses bien notées
```

### Visualisation complète

```
[Modèle pré-entraîné]
        ↓ SFT
[Modèle SFT]
        ↓
        ├── Génère des réponses
        ↓
[Annotateurs humains comparent les réponses A vs B]
        ↓
[Reward Model entraîné sur ces préférences]
        ↓
[PPO : optimise le modèle SFT selon le Reward Model]
        ↓
[Modèle aligné RLHF]
```

### Forces et limites du RLHF

```
✅ Résultats excellents — c'est ce qui a rendu ChatGPT si bon
✅ Apprend des nuances subjectives (clarté, honnêteté, utilité)
✅ Peut corriger des biais du SFT

❌ Très coûteux en annotation humaine (milliers d'heures)
❌ Complexe à implémenter (3 étapes, 3 modèles)
❌ Reward hacking : le modèle optimise le score sans vraiment s'améliorer
❌ Réservé aux équipes avec de gros budgets
```

---

## RLAIF — Reinforcement Learning from AI Feedback

Variante de RLHF où le feedback humain est remplacé par un LLM évaluateur.

```
RLHF  : Humains comparent A vs B → préférences humaines
RLAIF : LLM puissant compare A vs B → préférences du LLM

Avantage : 1000× moins cher, 1000× plus rapide
Limite   : la qualité dépend de la qualité du LLM évaluateur
```

### Mise en pratique RLAIF

```python
def comparer_réponses_rlaif(question: str, réponse_a: str, réponse_b: str) -> str:
    """Retourne 'A' ou 'B' selon le LLM évaluateur."""
    
    prompt = f"""Tu es un évaluateur expert en qualité de réponses LLM.

Question : {question}

Réponse A : {réponse_a}

Réponse B : {réponse_b}

Évalue selon ces critères :
1. Exactitude factuelle
2. Clarté et lisibilité
3. Utilité pour l'utilisateur
4. Concision (pas de remplissage)

Réponds UNIQUEMENT par "A" ou "B" (la meilleure réponse).
Si égalité parfaite, réponds "A"."""

    choix = llm_évaluateur.invoke(prompt).strip()
    return choix

# Générer un dataset de préférences avec RLAIF
dataset_préférences = []

for question in questions:
    réponse_a = modèle_candidat.invoke(question)
    réponse_b = modèle_candidat.invoke(question + " [variante]")
    
    préférence = comparer_réponses_rlaif(question, réponse_a, réponse_b)
    
    dataset_préférences.append({
        "prompt": question,
        "chosen": réponse_a if préférence == "A" else réponse_b,
        "rejected": réponse_b if préférence == "A" else réponse_a
    })
```

## DPO — Direct Preference Optimization

Alternative plus simple à RLHF/RLAIF. Pas besoin de Reward Model séparé.

```
RLHF : 3 étapes séparées (SFT → Reward Model → PPO)
DPO  : 1 seule étape d'optimisation directe

DPO prend en entrée :
  - Un dataset de paires (réponse choisie, réponse rejetée)
  - Optimise directement le modèle pour préférer "chosen" à "rejected"
```

```python
from trl import DPOTrainer, DPOConfig

config = DPOConfig(
    beta=0.1,  # régularisation KL (garde le modèle proche de la base)
    max_length=1024,
    per_device_train_batch_size=4,
    num_train_epochs=3,
    output_dir="./dpo_model"
)

trainer = DPOTrainer(
    model=modèle_sft,
    ref_model=modèle_référence,  # modèle de base non modifié
    args=config,
    train_dataset=dataset_préférences,
    tokenizer=tokenizer
)

trainer.train()
```

## Comparaison des approches d'alignement

| Approche | Complexité | Coût | Qualité | Usage typique |
|---|---|---|---|---|
| **SFT seul** | Faible | Faible | Bonne | La plupart des cas |
| **RLHF** | Très élevée | Très élevé | Excellente | Labs IA (OpenAI, Anthropic) |
| **RLAIF** | Élevée | Moyen | Très bonne | Startups avec ressources |
| **DPO** | Moyenne | Moyen | Très bonne | Alternative accessible à RLHF |

> [!tip] Pour la plupart des use cases
> SFT (Supervised Fine-Tuning) + un bon dataset suffit. RLHF/RLAIF/DPO apportent une valeur marginale si ton dataset SFT est déjà de haute qualité.

> [!info] DPO est la tendance actuelle
> DPO a largement remplacé RLHF dans les projets de fine-tuning car il est beaucoup plus simple à implémenter tout en donnant des résultats comparables.
