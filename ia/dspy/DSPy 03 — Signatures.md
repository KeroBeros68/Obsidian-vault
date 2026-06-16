#ia #dspy #signatures #bases

## Signatures

Une signature est la déclaration de ce qu'un module LLM doit faire — ses entrées et sorties. DSPy génère automatiquement le prompt à partir de cette déclaration.

## Syntaxe inline (simple)

```python
import dspy

# Format : "champ_input, ... -> champ_output, ..."
"question -> réponse"
"texte -> résumé"
"texte, langue -> traduction"
"question, contexte -> réponse, sources_utilisées"
```

## Syntaxe de classe (complète — recommandée)

```python
import dspy

class ClassificationSentiment(dspy.Signature):
    """Analyse le sentiment d'un texte client et identifie les sujets principaux."""

    # Champs d'entrée
    texte: str = dspy.InputField(desc="Le texte du client à analyser")

    # Champs de sortie
    sentiment: str = dspy.OutputField(
        desc="Le sentiment : 'positif', 'négatif' ou 'neutre'"
    )
    sujets: list[str] = dspy.OutputField(
        desc="Les 2-3 sujets principaux mentionnés dans le texte"
    )
    score_confiance: float = dspy.OutputField(
        desc="Score de confiance entre 0 et 1"
    )
```

## Types supportés dans les signatures

```python
import dspy
from typing import Literal, Optional

class ExempleTypage(dspy.Signature):
    """Montre les types supportés dans les signatures DSPy."""

    # Types basiques
    texte: str = dspy.InputField()
    nombre: int = dspy.InputField()
    score: float = dspy.InputField()
    actif: bool = dspy.InputField()

    # Types avancés
    catégorie: Literal["A", "B", "C"] = dspy.OutputField(
        desc="La catégorie, obligatoirement l'une de : A, B ou C"
    )
    tags: list[str] = dspy.OutputField(desc="Liste de tags")
    optionnel: Optional[str] = dspy.OutputField(desc="Peut être None")

    # Images (DSPy 2.5+ multimodal)
    image: dspy.Image = dspy.InputField(desc="Image à analyser")
```

## La docstring — le vrai prompt de haut niveau

La docstring de la classe signature sert d'instruction principale au LLM. C'est là que tu décris la tâche en langage naturel.

```python
class AnalyseTicketSupport(dspy.Signature):
    """
    Analyse un ticket de support client.
    Détermine la priorité (urgence), la catégorie du problème,
    et génère une réponse professionnelle et empathique.
    Ne jamais promettre de remboursement sans vérification.
    Toujours proposer une solution alternative si la demande est irréalisable.
    """

    ticket: str = dspy.InputField(desc="Le texte du ticket client")
    historique: str = dspy.InputField(
        desc="Historique des échanges précédents, vide si premier contact"
    )

    priorité: Literal["urgente", "haute", "normale", "faible"] = dspy.OutputField(
        desc="Niveau de priorité du ticket"
    )
    catégorie: str = dspy.OutputField(
        desc="Catégorie : livraison, paiement, retour, technique, autre"
    )
    réponse: str = dspy.OutputField(
        desc="Réponse professionnelle et empathique au client, 3-4 phrases max"
    )
```

## Utiliser une signature dans un module

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Syntaxe inline
classifier = dspy.Predict("texte -> sentiment, catégorie")
résultat = classifier(texte="Le colis n'est pas arrivé après 10 jours !")
print(résultat.sentiment)   # → "négatif"
print(résultat.catégorie)   # → "livraison"

# Syntaxe classe
classifier_complet = dspy.Predict(AnalyseTicketSupport)
résultat = classifier_complet(
    ticket="Mon colis n'est pas arrivé !",
    historique=""
)
print(résultat.priorité)   # → "haute"
print(résultat.réponse)    # → "Nous sommes désolés pour ce désagrément..."
```

## Composition de signatures

```python
import dspy

# Les signatures peuvent être héritées pour être étendues
class SignatureBase(dspy.Signature):
    """Analyse un texte."""
    texte: str = dspy.InputField()
    résumé: str = dspy.OutputField(desc="Résumé en 1 phrase")

class SignatureÉtendue(SignatureBase):
    """Analyse un texte et évalue sa tonalité."""
    tonalité: str = dspy.OutputField(desc="Formelle, informelle, ou technique")
    niveau_lecture: int = dspy.OutputField(desc="Niveau de lecture de 1 à 10")
```

## Ce que DSPy génère automatiquement

Quand tu utilises `dspy.Predict("question -> réponse")`, DSPy crée automatiquement un prompt structuré de type :

```
System: Your input fields are:
1. `question` (str)

Your output fields are:
1. `réponse` (str)

All interactions will be structured with the following input/output fields:
[INPUT]
Question: {question}

[OUTPUT]
Réponse: {réponse}
```

Après optimisation avec MIPROv2, ce prompt sera enrichi d'instructions et d'exemples few-shot générés automatiquement.

> [!tip] La docstring est le levier principal de qualité
> Avant d'optimiser avec un optimiseur, améliore d'abord la docstring de ta signature. Une description précise de la tâche, des contraintes et du format attendu améliore significativement la qualité sans coût d'optimisation.

> [!warning] Nommer les champs de façon descriptive
> Les noms des champs font partie du prompt. `réponse_empathique` est mieux que `output`. `ticket_client` est mieux que `input`. DSPy utilise ces noms pour guider le LLM.
