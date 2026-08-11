#ia #constrained-decoding #outlines #structured-generation #pratique

## Outlines — structured generation

Outlines (par dottxt-ai, ex-normal) est la librairie de référence pour le constrained decoding. Elle garantit des sorties structurées directement pendant la génération via des FSM compilées depuis les contraintes.

## Installation

```bash
pip install outlines

# Avec support transformers (modèles locaux)
pip install outlines transformers torch

# Avec support vLLM
pip install outlines vllm
```

## Charger un modèle

```python
import outlines

# Modèle local via transformers
model = outlines.models.transformers(
    "mistralai/Mistral-7B-Instruct-v0.3",
    device="cuda",
    model_kwargs={"torch_dtype": "float16"}
)

# Via vLLM (production)
model = outlines.models.vllm(
    "mistralai/Mistral-7B-Instruct-v0.3"
)

# Via Ollama (local, simple)
model = outlines.models.ollama("mistral", "http://localhost:11434")

# Via API OpenAI-compatible (Claude, GPT, vLLM serveur...)
model = outlines.models.openai("gpt-4o-mini")
```

> [!info] Le port 11434 est celui d'Ollama en local
> Voir [[Ollama — Index des fiches]] pour l'installation et [[Ollama 05 — Sécurité réseau (OLLAMA_HOST & port 11434)]] avant d'exposer cette API au-delà de la machine locale.

## Type 1 — Types de base

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Générer un entier
générateur = outlines.generate.format(model, int)
age = générateur("Quel est l'âge moyen d'un étudiant en master ?")
print(age)          # → 23  (garanti int, pas "vingt-trois")
print(type(age))    # → <class 'int'>

# Générer un float
générateur_float = outlines.generate.format(model, float)
score = générateur_float("Quel est le score moyen d'un bon modèle RAG ?")
print(score)   # → 0.87  (garanti float)

# Générer un bool
générateur_bool = outlines.generate.format(model, bool)
résultat = générateur_bool("Est-ce que Paris est la capitale de la France ?")
print(résultat)   # → True  (garanti bool)
```

## Type 2 — Choix (choice)

```python
import outlines
from typing import Literal

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Choix parmi une liste fixe
générateur = outlines.generate.choice(
    model,
    ["livraison", "retour", "paiement", "garantie", "autre"]
)

catégorie = générateur(
    "Classifie ce ticket : 'Mon colis n'est pas arrivé.'"
)
print(catégorie)   # → "livraison"  (TOUJOURS l'une des 5 options)

# Equivalent avec Literal
from outlines.generate import text
générateur_lit = outlines.generate.format(
    model,
    Literal["livraison", "retour", "paiement", "garantie", "autre"]
)
```

## Type 3 — Regex

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Date au format JJ/MM/AAAA
générateur_date = outlines.generate.regex(
    model,
    r"(0[1-9]|[12][0-9]|3[01])/(0[1-9]|1[012])/[12][0-9]{3}"
)
date = générateur_date("Quelle est la date de la Révolution Française ?")
print(date)   # → "14/07/1789"  (format garanti)

# Adresse IP
générateur_ip = outlines.generate.regex(
    model,
    r"((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"
)
ip = générateur_ip("Quelle est l'adresse IP de Google DNS ?")
print(ip)   # → "8.8.8.8"  (IP valide garantie)

# Code postal français
générateur_cp = outlines.generate.regex(model, r"[0-9]{5}")
cp = générateur_cp("Code postal de Paris ?")
print(cp)   # → "75001"
```

## Type 4 — JSON Schema / Pydantic (le plus utilisé)

```python
import outlines
from pydantic import BaseModel, Field
from typing import List, Literal, Optional

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Définir le schéma avec Pydantic
class AnalyseTicket(BaseModel):
    catégorie: Literal["livraison", "retour", "paiement", "garantie", "autre"]
    priorité:  Literal["urgente", "haute", "normale", "faible"]
    sentiment: Literal["positif", "négatif", "neutre"]
    résumé:    str = Field(description="Résumé en 1 phrase")
    actions:   List[str] = Field(description="Actions à effectuer")

# Créer le générateur structuré
générateur_json = outlines.generate.json(model, AnalyseTicket)

# Générer — résultat garanti conforme au schéma Pydantic
prompt = """Analyse ce ticket support :
"Bonjour, j'ai commandé un téléphone il y a 2 semaines et je n'ai toujours
rien reçu. C'est inadmissible ! Je veux être remboursé immédiatement."

Retourne une analyse structurée."""

résultat = générateur_json(prompt)

print(type(résultat))          # → <class 'AnalyseTicket'>
print(résultat.catégorie)      # → "livraison"
print(résultat.priorité)       # → "haute"
print(résultat.sentiment)      # → "négatif"
print(résultat.résumé)         # → "Client mécontent suite à retard de livraison..."
print(résultat.actions)        # → ["Vérifier le statut de livraison", "Contacter le transporteur"]
```

## Type 5 — Grammaire Context-Free (CFG)

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Grammaire EBNF pour des expressions arithmétiques simples
grammaire_arithmétique = r"""
    start: expr
    expr: term (("+" | "-") term)*
    term: factor (("*" | "/") factor)*
    factor: NUMBER | "(" expr ")"
    NUMBER: /[0-9]+/
    %ignore /\s+/
"""

générateur_expr = outlines.generate.cfg(model, grammaire_arithmétique)

expression = générateur_expr("Génère une expression arithmétique avec 3 et 4")
print(expression)   # → "3 + 4 * 2"  (expression valide garantie)

# Grammaire SQL simplifiée
grammaire_sql = r"""
    start: select_stmt
    select_stmt: "SELECT" columns "FROM" table
    columns: "*" | column ("," column)*
    column: /[a-zA-Z_][a-zA-Z0-9_]*/
    table: /[a-zA-Z_][a-zA-Z0-9_]*/
    %ignore /\s+/
"""

générateur_sql = outlines.generate.cfg(model, grammaire_sql)
query = générateur_sql("Génère une requête pour récupérer tous les utilisateurs")
print(query)   # → "SELECT * FROM utilisateurs"
```

## Génération en batch

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")
générateur = outlines.generate.json(model, AnalyseTicket)

# Traiter plusieurs tickets en parallèle
tickets = [
    "Mon colis n'est pas arrivé.",
    "Je veux retourner mon article défectueux.",
    "Ma carte bancaire est refusée."
]

prompts = [f"Analyse ce ticket : '{t}'" for t in tickets]

# Batch generation
résultats = générateur(prompts)   # liste de AnalyseTicket
for ticket, résultat in zip(tickets, résultats):
    print(f"{ticket[:30]:30} → {résultat.catégorie} / {résultat.priorité}")
```

## Réutiliser le générateur compilé

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Compiler une fois au démarrage (coût initial ~2-5s)
générateur = outlines.generate.json(model, AnalyseTicket)

# Réutiliser autant de fois que nécessaire (coût = microseconds)
for ticket in tickets_production:
    résultat = générateur(f"Analyse : '{ticket}'")
    # → Chaque génération est quasi-instantanée après la 1ère
```

> [!tip] Outlines suit le pattern Python natif
> `outlines.generate.json(model, MonSchéma)` suit exactement le système de types Python. Si ton type est `int`, tu obtiens un `int`. Si ton type est `Pydantic`, tu obtiens une instance Pydantic. Zéro parsing manuel.

> [!warning] Outlines ne gère pas les chat templates automatiquement
> Pour les modèles instruction-tuned (Mistral-Instruct, LLaMA-Instruct...), applique manuellement le chat template avant de passer le prompt à Outlines. Utilise `tokenizer.apply_chat_template()` puis passe la string résultante.
