#ia #dspy #fine-tuning #bootstrap-finetune #avancé

## BootstrapFinetune — fine-tuner les poids du LLM

Quand l'optimisation des prompts (MIPROv2, GEPA) atteint ses limites, `BootstrapFinetune` va plus loin en fine-tunant les poids du LLM lui-même à partir des traces générées par le programme.

## Principe

```
BootstrapFewShot / MIPROv2 :
  → Améliore les PROMPTS (instructions + exemples)
  → Le modèle reste identique
  → Limites : context window, coût par token

BootstrapFinetune :
  → Génère des traces (input → output réussis) depuis les prompts
  → Fine-tune le LLM sur ces traces
  → Le comportement est "gravé" dans les poids
  → Prompt plus court → moins de tokens → moins cher à l'inférence
```

## Pipeline complet BootstrapFinetune

```python
import dspy

# 1. Configurer le LM de base (pour générer les traces)
lm_base = dspy.LM("openai/gpt-4o-mini")
dspy.configure(lm=lm_base)

# 2. Ton programme à fine-tuner
class ClassifieurTicket(dspy.Module):
    def __init__(self):
        self.classifieur = dspy.ChainOfThought(
            "texte -> catégorie, priorité, réponse_suggérée"
        )

    def forward(self, texte: str):
        return self.classifieur(texte=texte)

# 3. Métrique
def métrique(exemple, prédiction, trace=None) -> bool:
    return (exemple.catégorie == prédiction.catégorie and
            exemple.priorité  == prédiction.priorité)

# 4. BootstrapFinetune
optimiseur = dspy.BootstrapFinetune(
    metric=métrique,
    num_threads=4
)

# 5. Compiler — génère des traces et lance le fine-tuning
programme_ft = optimiseur.compile(
    ClassifieurTicket(),
    trainset=trainset,
    # Le fine-tuning se fait sur le LM configuré (ici gpt-4o-mini)
    # Le modèle fine-tuné est automatiquement uploadé sur la plateforme du provider
)

# 6. Évaluer le programme fine-tuné
évaluateur = dspy.Evaluate(devset=devset, metric=métrique)
score = évaluateur(programme_ft)
print(f"Score avec fine-tuning: {score:.1%}")

# 7. Sauvegarder
programme_ft.save("./classifieur_ft.json")
```

## Stratégie combinée : MIPROv2 → BootstrapFinetune

La meilleure approche en production : optimiser d'abord les prompts, puis fine-tuner sur les meilleures traces.

```python
import dspy

dspy.configure(lm=dspy.LM("openai/gpt-4o-mini"))

# Étape 1 : Optimiser les prompts avec MIPROv2
opt_mipro = dspy.MIPROv2(metric=métrique, auto="medium")
programme_mipro = opt_mipro.compile(ClassifieurTicket(), trainset=trainset)

score_mipro = évaluateur(programme_mipro)
print(f"MIPROv2 : {score_mipro:.1%}")

# Étape 2 : Fine-tuner sur les traces du programme MIPROv2 optimisé
opt_ft = dspy.BootstrapFinetune(metric=métrique)
programme_final = opt_ft.compile(
    programme_mipro,   # partir du programme déjà optimisé par MIPROv2
    trainset=trainset
)

score_final = évaluateur(programme_final)
print(f"MIPROv2 + FineTune : {score_final:.1%}")

# Résultats typiques sur un vrai dataset :
# Baseline        : 61%
# MIPROv2         : 83%
# MIPROv2 + FT    : 91%
```

## Avantages du fine-tuning via DSPy

```
✅ Le prompt devient très court → moins de tokens → -60-80% de coût en inférence
✅ Le comportement est stable → moins de variabilité entre les appels
✅ Aucune dépendance aux exemples few-shot dans le prompt
✅ Modèle plus petit peut atteindre les performances d'un grand modèle

❌ Nécessite un dataset de qualité suffisant (50+ exemples)
❌ Temps de compilation plus long (fine-tuning = minutes à heures)
❌ Le modèle fine-tuné doit être hébergé
❌ Doit être ré-fine-tuné si la tâche évolue
```

## Utiliser un modèle local pour le fine-tuning

```python
import dspy

# Configurer vLLM comme backend
lm_local = dspy.LM(
    "openai/mistralai/Mistral-7B-Instruct-v0.3",
    api_base="http://localhost:8000/v1",
    api_key="no-key"
)
dspy.configure(lm=lm_local)

# BootstrapFinetune avec modèle local
# → Les traces sont générées localement
# → Le fine-tuning se fait avec transformers + PEFT
optimiseur = dspy.BootstrapFinetune(
    metric=métrique,
    num_threads=2
)

programme_ft_local = optimiseur.compile(ClassifieurTicket(), trainset=trainset)
```

## Quand choisir BootstrapFinetune ?

```
✅ Utiliser quand :
  - MIPROv2 a atteint ses limites et le score stagne
  - Tu as besoin de réduire les coûts en production (long terme)
  - Tu veux un petit modèle aussi performant qu'un grand
  - La tâche est très spécialisée et nécessite un comportement très stable

❌ Ne pas utiliser quand :
  - Tu as < 50 exemples de qualité
  - La tâche évolue souvent (le modèle doit être ré-fine-tuné)
  - Tu n'as pas les ressources pour héberger un modèle custom
  - Tu es encore en phase de prototypage
```

> [!tip] Ordre recommandé de progression
> 1. Prototype avec `Predict` sans optimisation
> 2. Optimise avec `BootstrapFewShot` (rapide, peu coûteux)
> 3. Améliore avec `MIPROv2(auto="medium")` (standard prod)
> 4. Fine-tune avec `BootstrapFinetune` si besoin de performance ou de réduction de coûts

> [!info] DSPy gère automatiquement le format des données d'entraînement
> BootstrapFinetune convertit automatiquement les traces DSPy au format JSONL pour le fine-tuning (format instruction/réponse). Tu n'as pas à préparer manuellement le dataset de fine-tuning.
