#ia #dspy #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier .with_inputs() sur les exemples

```python
import dspy

# ❌ Sans with_inputs() → DSPy ne sait pas quels champs sont les inputs
exemple = dspy.Example(
    question="Quel est le délai de retour ?",
    réponse="30 jours"
)
# → L'optimiseur traite TOUS les champs comme des inputs potentiels
# → Comportement indéfini pendant l'optimisation

# ✅ Toujours spécifier les inputs
exemple = dspy.Example(
    question="Quel est le délai de retour ?",
    réponse="30 jours"
).with_inputs("question")   # ← "question" est l'input, "réponse" est le label
```

---

## 🪤 Piège 2 — Métrique qui retourne toujours True

```python
import dspy

# ❌ Métrique trop permissive → l'optimiseur "converge" sans amélioration réelle
def mauvaise_métrique(exemple, prédiction, trace=None) -> bool:
    return len(prédiction.réponse) > 0   # toujours True si réponse non vide

# → L'optimiseur croit que tout est parfait dès le début
# → Aucune optimisation réelle

# ✅ Métrique discriminante
def bonne_métrique(exemple, prédiction, trace=None) -> float:
    attendu = exemple.réponse.lower()
    prédit  = prédiction.réponse.lower()
    # Score 1.0 seulement si le contenu attendu est présent
    if attendu in prédit:
        return 1.0
    # Score partiel si au moins 50% de correspondance
    mots_attendus = set(attendu.split())
    mots_prédits  = set(prédit.split())
    overlap = len(mots_attendus & mots_prédits) / max(len(mots_attendus), 1)
    return overlap if overlap > 0.5 else 0.0
```

---

## 🪤 Piège 3 — Évaluer sur le trainset

```python
import dspy

# ❌ Évaluer sur les exemples vus pendant l'optimisation → biais
opt = dspy.BootstrapFewShot(metric=métrique)
prog_optimisé = opt.compile(mon_prog, trainset=trainset)

score = dspy.Evaluate(devset=trainset, metric=métrique)(prog_optimisé)  # ❌ trainset !
# → Score gonflé artificiellement, ne reflète pas les vraies performances

# ✅ Toujours évaluer sur un devset/testset jamais vu pendant l'optimisation
score = dspy.Evaluate(devset=testset, metric=métrique)(prog_optimisé)
```

---

## 🪤 Piège 4 — Ne pas inspecter les prompts générés

```python
import dspy

# ❌ DSPy fait quelque chose d'inattendu, tu ne sais pas pourquoi
résultat = mon_module(question="...")
# Mauvaise réponse → "pourquoi ?"

# ✅ Inspecter ce que DSPy envoie réellement au LLM
résultat = mon_module(question="...")
dspy.inspect_history(n=1)
# → Affiche le prompt complet avec les instructions et exemples few-shot
# → Révèle souvent le problème (mauvaise instruction, exemple contre-productif...)
```

---

## 🪤 Piège 5 — Signature avec noms de champs trop génériques

```python
import dspy

# ❌ Noms génériques → le LLM ne sait pas quoi faire
class MauvaiseSig(dspy.Signature):
    """Traite l'input."""
    input: str  = dspy.InputField()
    output: str = dspy.OutputField()

# → "input" et "output" ne donnent aucune information au LLM

# ✅ Noms descriptifs + descriptions claires
class BonneSig(dspy.Signature):
    """Classifie un ticket de support client."""
    texte_ticket:    str = dspy.InputField(desc="Le message brut du client")
    catégorie:       str = dspy.OutputField(desc="Une de : livraison, retour, paiement, garantie, autre")
    niveau_urgence:  str = dspy.OutputField(desc="Une de : urgente, haute, normale, faible")
```

---

## 🪤 Piège 6 — Oublier de sauvegarder après optimisation

```python
import dspy

# ❌ Programme optimisé en mémoire seulement → perdu au redémarrage
opt = dspy.MIPROv2(metric=métrique, auto="medium")
prog = opt.compile(MonProgramme(), trainset=trainset)
# Redémarrage du programme → tous les prompts optimisés perdus !

# ✅ Sauvegarder immédiatement après optimisation
prog.save("./mon_programme_optimisé.json")

# Recharger au démarrage suivant
prog_rechargé = MonProgramme()
prog_rechargé.load("./mon_programme_optimisé.json")
```

---

## 🪤 Piège 7 — Utiliser Assert sur des tâches non déterministes

```python
import dspy

# ❌ Assert sur le contenu exact d'une réponse créative → retries infinis
class RédacteurCreatif(dspy.Module):
    def __init__(self):
        self.rédacteur = dspy.Predict("sujet -> article")

    def forward(self, sujet: str):
        résultat = self.rédacteur(sujet=sujet)

        # Mauvaise idée : la longueur exacte d'un article créatif est variable
        dspy.Assert(
            len(résultat.article) == 500,   # ← impossible à garantir exactement
            "L'article doit faire exactement 500 caractères."
        )
        return résultat

# ✅ Assert sur des propriétés vérifiables et atteignables
dspy.Assert(
    400 <= len(résultat.article) <= 600,
    "L'article doit faire entre 400 et 600 caractères."
)
```

---

## 🪤 Piège 8 — Dataset trop petit pour MIPROv2

```python
import dspy

# ❌ MIPROv2 avec seulement 10 exemples → résultats peu fiables
trainset_trop_petit = [dspy.Example(...).with_inputs("texte") for _ in range(10)]

opt = dspy.MIPROv2(metric=métrique, auto="medium")
prog = opt.compile(MonProgramme(), trainset=trainset_trop_petit)
# → L'optimiseur overfit sur 10 exemples → mauvaise généralisation

# ✅ Minimum recommandé par optimiseur :
# BootstrapFewShot : 20-50 exemples
# MIPROv2          : 50-200 exemples
# GEPA             : 100+ exemples
# BootstrapFinetune: 50+ exemples (pour des traces de qualité)
```

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Oublier `.with_inputs()` | Toujours `.with_inputs("champ1", "champ2")` sur chaque exemple |
| Métrique trop permissive | Vérifier que la métrique discrimine bien (score < 100% sur le baseline) |
| Évaluer sur le trainset | Toujours un `devset` / `testset` séparé pour l'évaluation finale |
| Ne pas inspecter les prompts | `dspy.inspect_history(n=1)` au moindre comportement inattendu |
| Noms de champs génériques | Noms descriptifs + `desc=` détaillé sur chaque champ |
| Ne pas sauvegarder | `.save()` immédiatement après chaque optimisation |
| Assert sur contenu variable | Assert sur des propriétés mesurables (longueur, présence d'un mot...) |
| Dataset trop petit | 50+ exemples pour MIPROv2, 20+ pour BootstrapFewShot |
