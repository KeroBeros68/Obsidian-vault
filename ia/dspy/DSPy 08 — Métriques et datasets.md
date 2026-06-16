#ia #dspy #métriques #datasets #évaluation #pratique

## Métriques et datasets

Les métriques et datasets sont le carburant des optimiseurs DSPy. Sans eux, l'optimisation automatique est impossible.

## Créer un dataset DSPy

```python
import dspy

# dspy.Example : un exemple d'entraînement ou d'évaluation
exemple = dspy.Example(
    question="Quel est le délai de retour ?",
    réponse_attendue="30 jours après réception",
    catégorie="politique_retour"
)

# Définir les champs d'entrée (ce qui est donné au module)
exemple_avec_inputs = exemple.with_inputs("question")
# → "question" est l'input, "réponse_attendue" et "catégorie" sont les labels

# Accéder aux champs
print(exemple.question)           # → "Quel est le délai de retour ?"
print(exemple.réponse_attendue)   # → "30 jours après réception"
print(exemple.inputs())           # → {"question": "..."}
print(exemple.labels())           # → {"réponse_attendue": "...", "catégorie": "..."}
```

## Construire un dataset complet

```python
import dspy

# Dataset de classification de tickets
dataset_raw = [
    {"texte": "Mon colis n'est pas arrivé.",    "catégorie": "livraison",  "priorité": "haute"},
    {"texte": "Je veux retourner mon article.", "catégorie": "retour",     "priorité": "normale"},
    {"texte": "Ma carte est refusée.",           "catégorie": "paiement",   "priorité": "urgente"},
    {"texte": "L'article est défectueux.",       "catégorie": "retour",     "priorité": "haute"},
    {"texte": "Livraison prévue quand ?",        "catégorie": "livraison",  "priorité": "faible"},
    # ... 50-500 exemples idéalement
]

# Convertir en dspy.Example
dataset = [
    dspy.Example(**d).with_inputs("texte")
    for d in dataset_raw
]

# Séparer en train / dev / test
import random
random.shuffle(dataset)
n = len(dataset)
trainset = dataset[:int(n * 0.6)]
devset   = dataset[int(n * 0.6):int(n * 0.8)]
testset  = dataset[int(n * 0.8):]

print(f"Train: {len(trainset)}, Dev: {len(devset)}, Test: {len(testset)}")
```

## Métriques — les différents types

### Métrique exacte (bool)

```python
def métrique_exacte(exemple, prédiction, trace=None) -> bool:
    """Vérifie si la catégorie prédite est exactement correcte."""
    return exemple.catégorie.lower() == prédiction.catégorie.lower()
```

### Métrique floue (float entre 0 et 1)

```python
def métrique_contient(exemple, prédiction, trace=None) -> float:
    """La réponse attendue doit être contenue dans la prédiction."""
    attendu = exemple.réponse_attendue.lower()
    prédit  = prédiction.réponse.lower()

    if attendu in prédit:
        return 1.0
    # Correspondance partielle — au moins la moitié des mots
    mots_attendus = set(attendu.split())
    mots_prédits  = set(prédit.split())
    overlap = len(mots_attendus & mots_prédits) / len(mots_attendus)
    return overlap
```

### Métrique LLM-as-Judge

```python
import dspy

class JugeQualité(dspy.Signature):
    """Évalue la qualité d'une réponse de support client."""
    question:         str   = dspy.InputField()
    réponse_générée:  str   = dspy.InputField()
    réponse_attendue: str   = dspy.InputField()
    score:            float = dspy.OutputField(desc="Score de 0 à 1")
    justification:    str   = dspy.OutputField()

juge = dspy.Predict(JugeQualité)

def métrique_llm_juge(exemple, prédiction, trace=None) -> float:
    """Utilise un LLM pour évaluer la qualité de la réponse."""
    évaluation = juge(
        question=exemple.question,
        réponse_générée=prédiction.réponse,
        réponse_attendue=exemple.réponse_attendue
    )
    return float(évaluation.score)
```

### Métrique composite (combinaison de critères)

```python
def métrique_composite(exemple, prédiction, trace=None) -> float:
    """Évalue plusieurs critères et retourne un score pondéré."""
    score = 0.0

    # Critère 1 : catégorie correcte (40%)
    if exemple.catégorie == prédiction.catégorie:
        score += 0.4

    # Critère 2 : priorité correcte (30%)
    if exemple.priorité == prédiction.priorité:
        score += 0.3

    # Critère 3 : réponse non vide et raisonnable (20%)
    if len(prédiction.réponse) > 20:
        score += 0.2

    # Critère 4 : pas de langage inapproprié (10%)
    mots_interdits = ["malheureusement", "désolé de vous informer", "impossible"]
    if not any(m in prédiction.réponse.lower() for m in mots_interdits):
        score += 0.1

    return score
```

## Évaluer un programme DSPy

```python
import dspy

évaluateur = dspy.Evaluate(
    devset=devset,
    metric=métrique_composite,
    num_threads=4,          # évaluation parallèle
    display_progress=True,  # barre de progression
    display_table=10        # afficher les 10 premiers exemples avec scores
)

# Évaluer avant optimisation
score_avant = évaluateur(mon_programme)
print(f"Score avant : {score_avant:.1%}")
```

## Charger un dataset HuggingFace

```python
import dspy
from datasets import load_dataset

# Charger un dataset HuggingFace
dataset_hf = load_dataset("clinc_oos", "plus")

# Convertir en dspy.Example
trainset_dspy = [
    dspy.Example(texte=ex["text"], catégorie=ex["intent"]).with_inputs("texte")
    for ex in dataset_hf["train"]
][:500]   # limiter pour les coûts
```

## Générer un dataset avec un LLM (bootstrapping)

```python
import dspy

générateur = dspy.Predict("domaine, catégorie -> texte_exemple, réponse_idéale")

dataset_généré = []
catégories = ["livraison", "retour", "paiement", "garantie"]
domaine = "support client e-commerce français"

for catégorie in catégories:
    for _ in range(25):   # 25 exemples par catégorie
        exemple = générateur(domaine=domaine, catégorie=catégorie)
        dataset_généré.append(
            dspy.Example(
                texte=exemple.texte_exemple,
                catégorie=catégorie,
                réponse_attendue=exemple.réponse_idéale
            ).with_inputs("texte")
        )

print(f"Dataset généré : {len(dataset_généré)} exemples")
```

> [!tip] Combien d'exemples pour les optimiseurs ?
> `BootstrapFewShot` : 20-50 exemples suffisent.
> `MIPROv2` : 50-200 exemples pour de bons résultats.
> `GEPA` : 100+ exemples recommandés.
> Qualité > quantité — 50 exemples bien construits > 500 exemples médiocres.

> [!warning] Ne jamais évaluer sur le trainset
> Les exemples du trainset sont vus pendant l'optimisation. Un bon score sur le trainset ne signifie rien. Toujours évaluer les performances finales sur le testset, jamais vu pendant l'optimisation.
