#ia #constrained-decoding #guidance #microsoft #dsl #avancé

## Guidance

Guidance (Microsoft) est un DSL Python qui entremêle génération et logique de contrôle dans le même code. Plutôt que de définir une contrainte séparée, tu écris un programme Python qui génère du texte de façon contrôlée.

## Installation

```bash
pip install guidance
```

## Concept — génération entrelacée

```python
import guidance
from guidance import models, gen, select, with_temperature

# Charger le modèle
lm = models.Transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Guidance entremêle texte fixe et génération
résultat = lm + f"""
Analyse ce ticket client : "Mon colis n'est pas arrivé."

Catégorie: {select(["livraison", "retour", "paiement", "autre"], name="catégorie")}
Priorité: {select(["urgente", "haute", "normale", "faible"], name="priorité")}
Résumé: {gen(name="résumé", max_tokens=50, stop="\n")}
Confiance: 0.{gen(name="confiance", regex=r"[0-9]{2}", max_tokens=2)}
"""

print(résultat["catégorie"])   # → "livraison"
print(résultat["priorité"])    # → "haute"
print(résultat["résumé"])      # → "Client n'a pas reçu son colis."
print(résultat["confiance"])   # → "89"
```

## Contrôle de flux avec Guidance

Guidance permet des structures conditionnelles dans la génération.

```python
import guidance
from guidance import models, gen, select, if_

lm = models.Transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Génération conditionnelle
@guidance
def analyser_ticket(lm, ticket: str):
    lm += f"Ticket: {ticket}\n"
    lm += f"Catégorie: {select(['livraison', 'retour', 'urgent'], name='cat')}\n"

    # Si urgent → champ supplémentaire
    if lm["cat"] == "urgent":
        lm += f"Escalader à: {gen(name='escalade', max_tokens=20, stop=chr(10))}\n"

    lm += f"Résumé: {gen(name='résumé', max_tokens=50, stop=chr(10))}\n"
    return lm

résultat = analyser_ticket(lm, "URGENT : client VIP insatisfait")
print(résultat["résumé"])
```

## Génération avec regex dans Guidance

```python
import guidance
from guidance import models, gen

lm = models.Transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Format structuré avec regex
résultat = lm + f"""
Extrais les informations de contact :
Texte: "Contactez Jean Dupont au 06 12 34 56 78 ou jean@example.com"

Nom: {gen(name="nom", regex=r"[A-Z][a-z]+ [A-Z][a-z]+")}
Téléphone: {gen(name="tel", regex=r"0[67] [0-9]{{2}} [0-9]{{2}} [0-9]{{2}} [0-9]{{2}}")}
Email: {gen(name="email", regex=r"[a-z]+@[a-z]+\.[a-z]+")}
"""

print(résultat["nom"])    # → "Jean Dupont"
print(résultat["tel"])    # → "06 12 34 56 78"
print(résultat["email"])  # → "jean@example.com"
```

## Guidance vs Outlines — philosophie différente

```
Outlines :
  Tu définis la CONTRAINTE (JSON Schema, regex)
  Outlines gère la génération entière
  → Séparation nette contrainte / génération
  → Idéal pour du JSON pur

Guidance :
  Tu écris un PROGRAMME qui génère du texte
  La logique Python est entremêlée avec la génération
  → Contrôle très fin étape par étape
  → Idéal pour des formats complexes avec logique conditionnelle
```

## Guidance avec OpenAI / Azure

```python
import guidance
from guidance import models, gen, select

# Azure OpenAI
lm = models.AzureOpenAI(
    model="gpt-4o",
    azure_endpoint="https://ton-endpoint.openai.azure.com",
    api_key="ta-clé"
)

résultat = lm + f"Catégorie du ticket: {select(['A', 'B', 'C'], name='cat')}"
print(résultat["cat"])
```

> [!tip] Guidance pour les workflows complexes
> Si ton pipeline nécessite de la logique conditionnelle (si X alors générer Y, sinon générer Z), Guidance est plus expressif qu'Outlines. Si tu as juste besoin d'un JSON valide, Outlines est plus simple.

> [!info] llguidance — le moteur Rust sous Guidance
> Guidance utilise `llguidance` comme moteur sous le capot. `llguidance` calcule les masques de tokens à la volée sans startup cost, ce qui le distingue d'Outlines (pré-calcul) et le rend très rapide.
