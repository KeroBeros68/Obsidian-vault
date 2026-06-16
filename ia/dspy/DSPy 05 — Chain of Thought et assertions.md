#ia #dspy #chain-of-thought #assertions #intermédiaire

## Chain of Thought et assertions

## ChainOfThought — raisonnement explicite

DSPy's `ChainOfThought` ajoute un champ `reasoning` qui force le LLM à réfléchir avant de répondre, sans que tu aies à écrire "réfléchis étape par étape" dans le prompt.

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Comparaison Predict vs ChainOfThought
predict_simple = dspy.Predict("question, contexte -> réponse")
cot_module     = dspy.ChainOfThought("question, contexte -> réponse")

contexte = """
Politique Acme Corp :
- Retours : 30 jours, emballage original requis
- Remboursement : 5-7 jours ouvrés
- Produits défectueux : repris sans frais, échange ou remboursement
- Exceptions : produits personnalisés non retournables
"""

question = "Mon colis est arrivé avec de la casse. Puis-je avoir un remboursement intégral ?"

# Avec Predict
r1 = predict_simple(question=question, contexte=contexte)
print("Predict:", r1.réponse)

# Avec ChainOfThought
r2 = cot_module(question=question, contexte=contexte)
print("Reasoning:", r2.reasoning)   # raisonnement interne visible
print("CoT:", r2.réponse)           # réponse finale mieux construite
```

## ChainOfThoughtWithHint — raisonnement guidé

Permet de fournir un indice (hint) pour orienter le raisonnement.

```python
import dspy

cot_hint = dspy.ChainOfThoughtWithHint("question -> réponse")

résultat = cot_hint(
    question="Quel est l'impact environnemental du numérique ?",
    hint="Penser aux data centers, à la fabrication des appareils, et à la consommation électrique."
)
print(résultat.réponse)
```

## Assertions — contraindre les sorties

Les assertions permettent de définir des règles que la sortie doit respecter. Si une règle est violée, DSPy relance automatiquement le LLM avec des instructions de correction.

### dspy.Assert — assertion stricte (lève une exception si violée)

```python
import dspy

class RépondeurContraint(dspy.Module):
    def __init__(self):
        self.répondeur = dspy.ChainOfThought("question -> réponse")

    def forward(self, question: str):
        résultat = self.répondeur(question=question)

        # La réponse doit faire moins de 100 mots
        dspy.Assert(
            len(résultat.réponse.split()) < 100,
            "La réponse doit faire moins de 100 mots. Sois plus concis."
        )

        # La réponse ne doit pas contenir de langage offensant
        dspy.Assert(
            "merde" not in résultat.réponse.lower(),
            "La réponse ne doit pas contenir de langage inapproprié."
        )

        return résultat
```

### dspy.Suggest — suggestion non-bloquante (log l'échec mais continue)

```python
import dspy

class RépondeurSouple(dspy.Module):
    def __init__(self):
        self.répondeur = dspy.ChainOfThought("question -> réponse")

    def forward(self, question: str):
        résultat = self.répondeur(question=question)

        # Suggestion : la réponse devrait mentionner les sources
        dspy.Suggest(
            "source" in résultat.réponse.lower() or "selon" in résultat.réponse.lower(),
            "Essaie de citer les sources ou documents qui justifient ta réponse."
        )

        # Suggestion : réponse structurée
        dspy.Suggest(
            any(marker in résultat.réponse for marker in ["1.", "•", "-", "\n"]),
            "Essaie de structurer ta réponse avec des points ou des listes."
        )

        return résultat
```

## Retry automatique avec les assertions

Quand une assertion `dspy.Assert` échoue, DSPy relance le LLM avec un message d'erreur enrichi. Par défaut : 1 retry.

```python
import dspy

# Configurer le nombre de retries global
dspy.configure(
    lm=dspy.LM("anthropic/claude-sonnet-4-20250514"),
    max_errors=3   # max 3 retries par assertion
)

class ClassifieurStrict(dspy.Module):
    def __init__(self):
        self.classifieur = dspy.Predict("texte -> catégorie")
        self.catégories_valides = {"retour", "livraison", "garantie", "paiement", "autre"}

    def forward(self, texte: str):
        résultat = self.classifieur(texte=texte)

        # Assertion : la catégorie doit être dans la liste valide
        dspy.Assert(
            résultat.catégorie.lower() in self.catégories_valides,
            f"La catégorie '{résultat.catégorie}' n'est pas valide. "
            f"Choisir parmi : {', '.join(self.catégories_valides)}"
        )

        return résultat

classifieur = ClassifieurStrict()
résultat = classifieur(texte="Je veux renvoyer mon article.")
print(résultat.catégorie)   # → "retour" (garanti dans la liste valide)
```

## Assertions avec validation Pydantic

```python
import dspy
from pydantic import BaseModel, field_validator

class AnalyseSentiment(BaseModel):
    sentiment: str
    score: float

    @field_validator("sentiment")
    def valider_sentiment(cls, v):
        if v not in ["positif", "négatif", "neutre"]:
            raise ValueError(f"Sentiment invalide: {v}")
        return v

    @field_validator("score")
    def valider_score(cls, v):
        if not 0 <= v <= 1:
            raise ValueError(f"Score doit être entre 0 et 1, reçu: {v}")
        return v

class AnalyseurSentiment(dspy.Module):
    def __init__(self):
        self.analyseur = dspy.Predict(
            "texte -> sentiment, score: float"
        )

    def forward(self, texte: str):
        résultat = self.analyseur(texte=texte)

        # Validation avec Pydantic
        try:
            analyse = AnalyseSentiment(
                sentiment=résultat.sentiment,
                score=résultat.score
            )
        except Exception as e:
            dspy.Assert(False, f"Sortie invalide : {e}")

        return résultat
```

> [!tip] Assert pour les champs critiques, Suggest pour les améliorations
> `dspy.Assert` garantit le respect d'une contrainte au prix d'un retry (coût supplémentaire). `dspy.Suggest` guide le LLM sans bloquer. Utilise `Assert` pour les règles métier critiques, `Suggest` pour améliorer la qualité.

> [!warning] Les assertions ont un coût
> Chaque retry suite à une assertion échouée = un appel LLM supplémentaire. Sur un système à fort volume, des assertions trop restrictives peuvent tripler le coût. Calibre les assertions selon l'importance de la contrainte vs le coût.
