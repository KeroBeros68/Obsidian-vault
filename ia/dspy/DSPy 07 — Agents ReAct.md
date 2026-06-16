#ia #dspy #agents #react #outils #intermédiaire

## Agents ReAct avec DSPy

DSPy's `dspy.ReAct` implémente le pattern Reasoning + Acting. La différence avec LangChain : le comportement de l'agent (quand et comment utiliser les outils) est optimisable automatiquement.

## Agent ReAct minimal

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

# Définir les outils — les docstrings sont critiques
def rechercher_docs(query: str) -> str:
    """Recherche dans la base documentaire interne.
    Utiliser pour : politiques, procédures, informations produit.
    Ne pas utiliser pour : actualités, prix temps réel."""
    # ... logique de recherche
    return f"Résultat pour '{query}': La politique de retour est de 30 jours..."

def calculer(expression: str) -> str:
    """Calcule une expression mathématique simple.
    Exemples valides: '30 * 2', '100 / 4 + 5'
    Ne pas utiliser pour des calculs de dates."""
    try:
        return str(eval(expression, {"__builtins__": {}}, {}))
    except Exception as e:
        return f"Erreur de calcul: {e}"

def vérifier_stock(référence_produit: str) -> str:
    """Vérifie le stock d'un produit par sa référence.
    Format référence: 'REF-XXXX'"""
    # ... appel API
    return f"Produit {référence_produit}: 15 unités en stock"

# Créer l'agent ReAct
agent = dspy.ReAct(
    "question -> réponse",
    tools=[rechercher_docs, calculer, vérifier_stock],
    max_iters=5   # maximum 5 iterations outil
)

résultat = agent(question="Est-ce que le produit REF-1234 est disponible et quel est le délai de retour ?")
print(résultat.réponse)
# → "Le produit REF-1234 est en stock (15 unités). La politique de retour est de 30 jours..."
```

## Agent avec signature complète

```python
import dspy

class AgentSignature(dspy.Signature):
    """Agent support client expert.
    Utilise les outils disponibles pour répondre précisément.
    Toujours vérifier les faits avant de répondre.
    Ne jamais inventer d'informations."""

    historique: list[dict] = dspy.InputField(desc="Historique de la conversation")
    question:   str        = dspy.InputField(desc="Question ou demande du client")
    réponse:    str        = dspy.OutputField(desc="Réponse professionnelle et précise")
    actions:    list[str]  = dspy.OutputField(desc="Actions internes à effectuer après cet échange")

agent = dspy.ReAct(AgentSignature, tools=[rechercher_docs, calculer, vérifier_stock])

résultat = agent(
    historique=[{"role": "user", "content": "J'ai commandé hier"}, {"role": "assistant", "content": "Bonjour !"}],
    question="Mon colis REF-5678 est-il en stock ?"
)
print(résultat.réponse)
print(résultat.actions)
```

## Agent custom avec logique de contrôle

```python
import dspy

class AgentAvancé(dspy.Module):
    def __init__(self, retriever):
        self.retriever = retriever

        # Outil de recherche wrappé
        def rechercher(query: str) -> str:
            """Recherche dans la base documentaire. Utiliser pour toute question factuelle."""
            passages = retriever(query)
            return "\n".join(passages[:3])

        def escalader(raison: str) -> str:
            """Escalader le ticket à un humain. Utiliser si : remboursement > 100€, plainte formelle, demande juridique."""
            return f"Ticket escaladé : {raison}. Un conseiller vous contactera sous 24h."

        self.agent = dspy.ReAct(
            "historique: list[str], question -> réponse, escaladé: bool",
            tools=[rechercher, calculer, escalader],
            max_iters=7
        )

    def forward(self, question: str, historique: list[str] = None):
        historique = historique or []

        résultat = self.agent(
            historique=historique,
            question=question
        )

        # Guardrail : vérifier si l'escalade est nécessaire
        dspy.Assert(
            isinstance(résultat.escaladé, bool),
            "Le champ 'escaladé' doit être un booléen"
        )

        return résultat
```

## Tracer les étapes de l'agent

```python
import dspy

dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-20250514"))

agent = dspy.ReAct("question -> réponse", tools=[rechercher_docs, calculer])

# Activer les traces pour voir toutes les étapes
with dspy.context(trace=[]):
    résultat = agent(question="Combien font 2 fois le délai de retour en jours ?")

# Inspecter les étapes (tool calls, reasoning...)
dspy.inspect_history(n=1)
# → Affiche chaque itération : Thought → Action → Observation → ...
```

## Agent multi-outils avec gestion des erreurs

```python
import dspy
from typing import Optional

def api_commandes(numéro_commande: str) -> str:
    """Récupère les détails d'une commande client.
    Format numéro: 'CMD-XXXXXX'
    Retourne le statut, la date et le montant."""
    try:
        # Appel API simulé
        if not numéro_commande.startswith("CMD-"):
            return "Erreur: format invalide. Utiliser CMD-XXXXXX"
        return f"Commande {numéro_commande}: Livrée le 10/05/2025, 89,90€"
    except Exception as e:
        return f"Erreur technique: {str(e)}"

def politique_remboursement(montant: float, motif: str) -> str:
    """Vérifie si un remboursement est autorisé selon la politique.
    Args: montant en euros, motif (défectueux/insatisfaction/erreur)"""
    if montant > 100 and motif != "défectueux":
        return "Remboursement > 100€ nécessite validation manager"
    return f"Remboursement de {montant}€ pour motif '{motif}' autorisé"

agent_complet = dspy.ReAct(
    "question -> réponse, prochaine_action: str",
    tools=[rechercher_docs, api_commandes, politique_remboursement, calculer],
    max_iters=6
)

résultat = agent_complet(
    question="Ma commande CMD-123456 est arrivée endommagée. Le montant était 75€. Puis-je être remboursé ?"
)
print(résultat.réponse)
print("Prochaine action:", résultat.prochaine_action)
```

## Optimiser l'agent

```python
import dspy

# Dataset d'exemples pour optimiser l'agent
trainset = [
    dspy.Example(
        question="Mon colis REF-001 est disponible ?",
        réponse="Oui, le produit REF-001 est disponible."
    ).with_inputs("question"),
    # ... autres exemples
]

def métrique_agent(exemple, prédiction, trace=None) -> float:
    # Vérifier que la réponse contient les infos attendues
    score = 0.0
    if exemple.réponse.lower() in prédiction.réponse.lower():
        score += 0.8
    if len(prédiction.réponse) > 20:  # réponse non vide
        score += 0.2
    return score

optimiseur = dspy.BootstrapFewShot(metric=métrique_agent, max_bootstrapped_demos=3)
agent_optimisé = optimiseur.compile(agent_complet, trainset=trainset)
agent_optimisé.save("./agent_optimisé.json")
```

> [!tip] max_iters selon la complexité
> Pour des questions simples : `max_iters=3`. Pour des workflows multi-étapes : `max_iters=7`. Au-delà de 10, l'agent risque de tourner en rond. Commence petit et augmente si nécessaire.

> [!info] DSPy optimise le comportement de l'agent
> Contrairement à LangChain où tu hardcodes "utilise d'abord cet outil puis cet autre", DSPy apprend automatiquement le meilleur ordre et la meilleure façon d'utiliser les outils grâce aux optimiseurs.
