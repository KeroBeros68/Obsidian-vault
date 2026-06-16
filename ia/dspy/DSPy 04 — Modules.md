#ia #dspy #modules #bases

## Modules

Les modules sont les briques d'exécution de DSPy. Ils définissent **comment** interroger le LLM pour réaliser une signature. Chaque module est composable et optimisable.

## Les modules built-in

### dspy.Predict — le module de base

Appel direct au LLM sans stratégie particulière.

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Création
classifieur = dspy.Predict("texte -> catégorie, sentiment")

# Appel
résultat = classifieur(texte="Mon colis n'est pas arrivé depuis 2 semaines !")
print(résultat.catégorie)  # → "livraison"
print(résultat.sentiment)  # → "négatif"

# Accéder aux métadonnées
print(résultat.completions)  # → liste des complétions brutes du LLM
```

### dspy.ChainOfThought — raisonnement étape par étape

Ajoute un champ `reasoning` caché qui force le LLM à réfléchir avant de répondre. Améliore significativement la qualité sur les tâches complexes.

```python
import dspy

# ChainOfThought ajoute automatiquement un champ "reasoning" interne
cot = dspy.ChainOfThought("question, contexte -> réponse")

résultat = cot(
    question="Quelle est la procédure de retour pour un article défectueux ?",
    contexte="Notre politique : retour sous 30 jours, article dans emballage original."
)

print(résultat.reasoning)  # → Le LLM montre son raisonnement
print(résultat.réponse)    # → La réponse finale, mieux construite
```

### dspy.ProgramOfThought — génère et exécute du code

Génère du code Python pour résoudre des problèmes, l'exécute et retourne le résultat.

```python
import dspy

pot = dspy.ProgramOfThought("question -> réponse")

résultat = pot(question="Quelle est la racine carrée de 144 × 25 + 100 ?")
print(résultat.code)     # → import math\nprint(math.sqrt(144*25+100))
print(résultat.réponse)  # → "60.0"
```

### dspy.ReAct — agent avec outils

Implémente le pattern ReAct (Reasoning + Acting) avec des outils.

```python
import dspy

def rechercher(query: str) -> str:
    """Recherche des informations dans la base documentaire."""
    return f"Résultat de recherche pour '{query}': ..."

def calculer(expression: str) -> str:
    """Calcule une expression mathématique."""
    return str(eval(expression, {"__builtins__": {}}, {}))

agent = dspy.ReAct(
    "question -> réponse",
    tools=[rechercher, calculer]
)

résultat = agent(question="Quel est le délai de retour multiplié par 2 ?")
print(résultat.réponse)
```

### dspy.MultiChainComparison — comparer plusieurs raisonnements

Génère plusieurs raisonnements et choisit le meilleur.

```python
import dspy

mcc = dspy.MultiChainComparison("question -> réponse", M=3)
# Génère 3 raisonnements ChainOfThought et sélectionne le plus cohérent
résultat = mcc(question="Question complexe...")
```

## Créer un module custom

```python
import dspy

class AnalyseurAvancé(dspy.Module):
    def __init__(self):
        # Déclarer les sous-modules
        self.classifieur = dspy.Predict("texte -> catégorie, priorité")
        self.répondeur   = dspy.ChainOfThought(
            "texte, catégorie, priorité -> réponse_client, actions_internes"
        )
        self.validateur  = dspy.Predict(
            "réponse_client -> réponse_validée, score_qualité: float"
        )

    def forward(self, texte: str) -> dspy.Prediction:
        # Étape 1 : classifier
        classif = self.classifieur(texte=texte)

        # Étape 2 : générer une réponse
        réponse = self.répondeur(
            texte=texte,
            catégorie=classif.catégorie,
            priorité=classif.priorité
        )

        # Étape 3 : valider la réponse
        validation = self.validateur(réponse_client=réponse.réponse_client)

        # Retourner une Prediction avec tous les champs utiles
        return dspy.Prediction(
            catégorie=classif.catégorie,
            priorité=classif.priorité,
            réponse=validation.réponse_validée,
            score=validation.score_qualité,
            actions=réponse.actions_internes
        )

# Utilisation
dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))
analyseur = AnalyseurAvancé()
résultat  = analyseur(texte="Mon colis est arrivé cassé, je veux un remboursement.")

print(résultat.catégorie)  # → "retour"
print(résultat.priorité)   # → "haute"
print(résultat.réponse)    # → "Nous sommes vraiment désolés..."
print(résultat.score)      # → 0.92
```

## Sauvegarder et charger un module

```python
import dspy

# Sauvegarder (avec les prompts optimisés si déjà compilé)
analyseur.save("./mon_analyseur.json")

# Recharger
analyseur_chargé = AnalyseurAvancé()
analyseur_chargé.load("./mon_analyseur.json")
```

## Composition de modules

```python
import dspy

class PipelineComplet(dspy.Module):
    """Pipeline : extraction → résumé → classification → réponse."""

    def __init__(self):
        self.extracteur   = dspy.Predict("document -> faits_clés: list[str]")
        self.résumeur     = dspy.ChainOfThought("faits_clés -> résumé")
        self.classifieur  = dspy.Predict("résumé -> catégorie, urgence")
        self.rédacteur    = dspy.ChainOfThought(
            "résumé, catégorie, urgence -> réponse_finale"
        )

    def forward(self, document: str):
        faits     = self.extracteur(document=document)
        résumé    = self.résumeur(faits_clés=faits.faits_clés)
        classif   = self.classifieur(résumé=résumé.résumé)
        réponse   = self.rédacteur(
            résumé=résumé.résumé,
            catégorie=classif.catégorie,
            urgence=classif.urgence
        )
        return réponse
```

## Tableau des modules disponibles

| Module | Stratégie | Meilleur pour |
|---|---|---|
| `Predict` | Appel direct | Tâches simples, classification |
| `ChainOfThought` | Raisonnement puis réponse | Tâches complexes, QA |
| `ProgramOfThought` | Génère et exécute du code | Calculs, problèmes formels |
| `ReAct` | Agent avec outils | Recherche, multi-étapes |
| `MultiChainComparison` | Vote entre raisonnements | Haute précision requise |
| Module custom | Composition libre | Tout pipeline spécifique |

> [!tip] ChainOfThought par défaut pour les tâches non triviales
> Sur toute tâche qui nécessite plus qu'une simple classification, utilise `ChainOfThought` plutôt que `Predict`. Le raisonnement explicite améliore la qualité sans coût supplémentaire significatif.

> [!warning] Les modules sont stateful après optimisation
> Quand tu optimises un module avec un optimiseur, ses prompts changent. Sauvegarde le module optimisé avec `.save()` avant de le déployer.
