#ia #dspy #optimiseurs #bootstrap #mipro #gepa #avancé

## Optimiseurs — BootstrapFewShot, MIPROv2, GEPA

Les optimiseurs sont le cœur de DSPy. Ils améliorent automatiquement les programmes en optimisant les prompts (instructions + exemples few-shot) selon une métrique définie.

## Vue d'ensemble des optimiseurs

```
BootstrapFewShot     : génère des exemples few-shot optimaux
                       Simple, peu coûteux, bon point de départ

BootstrapFewShotRS   : comme ci-dessus avec random search
                       Meilleur mais plus coûteux

MIPROv2              : optimise instructions + exemples simultanément
                       Standard recommandé en production

GEPA                 : optimisation génétique des instructions
                       Meilleur résultat, le plus coûteux

BootstrapFinetune    : fine-tune les poids du LLM (voir fiche 10)
                       Quand les prompts ne suffisent plus
```

## BootstrapFewShot — le plus simple

Génère des traces d'exécution réussies et les utilise comme exemples few-shot.

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Ton programme à optimiser
class ClassifieurTicket(dspy.Module):
    def __init__(self):
        self.classifieur = dspy.ChainOfThought(
            "texte -> catégorie, priorité, réponse_suggérée"
        )

    def forward(self, texte: str):
        return self.classifieur(texte=texte)

# Métrique d'évaluation
def métrique(exemple, prédiction, trace=None) -> bool:
    return (exemple.catégorie == prédiction.catégorie and
            exemple.priorité  == prédiction.priorité)

# Optimiser
optimiseur = dspy.BootstrapFewShot(
    metric=métrique,
    max_bootstrapped_demos=4,    # max exemples few-shot par module
    max_labeled_demos=4,         # max exemples labellisés
    max_errors=5                 # max erreurs avant abandon
)

programme_optimisé = optimiseur.compile(
    ClassifieurTicket(),
    trainset=trainset
)

# Évaluer
évaluateur = dspy.Evaluate(devset=devset, metric=métrique)
print(f"Score: {évaluateur(programme_optimisé):.1%}")

# Sauvegarder
programme_optimisé.save("./classifieur_optimisé.json")
```

## MIPROv2 — l'optimiseur standard de production

MIPROv2 commence par une phase de bootstrapping — il exécute ton programme sur différents inputs pour collecter des traces de comportement, puis propose et explore intelligemment de meilleures instructions en langage naturel pour chaque prompt.

```python
import dspy

# MIPROv2 optimise SIMULTANÉMENT :
# 1. Les instructions (docstring du prompt)
# 2. Les exemples few-shot
# Résultat : meilleur que BootstrapFewShot seul

optimiseur = dspy.MIPROv2(
    metric=métrique,
    auto="medium",              # "light", "medium", "heavy"
                                # light  : ~$0.50, ~5 min
                                # medium : ~$2, ~20 min (recommandé)
                                # heavy  : ~$10, ~1h
    num_threads=4               # appels parallèles
)

programme_mipro = optimiseur.compile(
    ClassifieurTicket(),
    trainset=trainset,
    num_trials=25,              # nombre de configurations explorées
    minibatch_size=25,          # taille du batch d'évaluation pendant l'optim
    minibatch_full_eval_steps=5 # évaluation complète tous les N trials
)

programme_mipro.save("./classifieur_mipro.json")
```

## GEPA — optimisation génétique avancée

GEPA (Generative Evolutionary Prompt Architect) utilise une approche génétique pour optimiser les instructions, avec support custom pour les instruction proposers et les modèles multimodaux.

```python
import dspy

# GEPA utilise des algorithmes évolutionnaires pour découvrir
# les meilleures instructions — plus puissant mais plus coûteux
optimiseur = dspy.GEPA(
    metric=métrique,
    num_threads=4
)

programme_gepa = optimiseur.compile(
    ClassifieurTicket(),
    trainset=trainset,
    devset=devset    # GEPA utilise un devset pour valider les mutations
)
```

## Comparaison pratique — même tâche, différents optimiseurs

```python
import dspy

résultats = {}

# Baseline (sans optimisation)
programme_base = ClassifieurTicket()
résultats["baseline"] = évaluateur(programme_base)

# BootstrapFewShot
opt_bootstrap = dspy.BootstrapFewShot(metric=métrique, max_bootstrapped_demos=4)
prog_bootstrap = opt_bootstrap.compile(ClassifieurTicket(), trainset=trainset)
résultats["bootstrap"] = évaluateur(prog_bootstrap)

# MIPROv2
opt_mipro = dspy.MIPROv2(metric=métrique, auto="medium")
prog_mipro = opt_mipro.compile(ClassifieurTicket(), trainset=trainset)
résultats["mipro"] = évaluateur(prog_mipro)

for nom, score in résultats.items():
    print(f"{nom:15} : {score:.1%}")

# Exemple de résultats typiques :
# baseline        : 61.0%
# bootstrap       : 74.0%
# mipro           : 83.0%
```

## Charger et réutiliser un programme optimisé

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Charger le programme optimisé
programme = ClassifieurTicket()
programme.load("./classifieur_mipro.json")

# Utiliser directement — les prompts optimisés sont chargés
résultat = programme(texte="Mon colis n'est pas arrivé depuis 2 semaines !")
print(résultat.catégorie)    # → "livraison"
print(résultat.priorité)     # → "haute"
```

## Inspecter ce que l'optimiseur a produit

```python
import dspy

# Voir les prompts optimisés générés
for nom, module in programme_mipro.named_predictors():
    print(f"\n── Module : {nom} ──")
    print("Instructions:", module.signature.__doc__)
    print("Nombre de demos:", len(module.demos))
    for i, demo in enumerate(module.demos[:2]):
        print(f"  Exemple {i+1}:", demo)
```

## Choisir son optimiseur

| Situation | Optimiseur recommandé |
|---|---|
| < 20 exemples, prototype rapide | `BootstrapFewShot` |
| 50-200 exemples, production | `MIPROv2(auto="medium")` |
| 100+ exemples, qualité maximale | `GEPA` ou `MIPROv2(auto="heavy")` |
| Amélioration insuffisante avec les prompts | `BootstrapFinetune` (voir fiche 10) |

> [!tip] Coût typique d'une optimisation MIPROv2
> Un run d'optimisation typique coûte environ $2 USD et prend ~20 minutes. Sois attentif si tu utilises de très grands LLM ou de très grands datasets.

> [!warning] L'optimisation n'est pas magique
> Si ta métrique est mal définie ou ton dataset non représentatif, l'optimisation peut produire un programme qui score bien sur le devset mais mal en production. Investis d'abord dans la qualité de ta métrique et de ton dataset.
