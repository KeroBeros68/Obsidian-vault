#ia #constrained-decoding #vllm #production #pratique

## Constrained Decoding avec vLLM

vLLM supporte nativement le constrained decoding via trois backends : Outlines, LM Format Enforcer, et XGrammar. C'est l'approche recommandée en production.

## Lancer vLLM avec guided decoding

```bash
# Lancer vLLM — le guided decoding est activé par défaut
python -m vllm.entrypoints.openai.api_server \
    --model mistralai/Mistral-7B-Instruct-v0.3 \
    --port 8000

# Spécifier le backend par défaut (xgrammar recommandé en prod)
python -m vllm.entrypoints.openai.api_server \
    --model mistralai/Mistral-7B-Instruct-v0.3 \
    --guided-decoding-backend xgrammar \
    --port 8000
```

## Les paramètres guided decoding de vLLM

vLLM supporte la génération de sorties structurées via les paramètres suivants dans l'API OpenAI : `guided_choice`, `guided_regex`, `guided_json`, `guided_grammar`, et `guided_decoding_backend`.

```python
from openai import OpenAI
from pydantic import BaseModel
from typing import Literal, List
import json

client = OpenAI(base_url="http://localhost:8000/v1", api_key="no-key")

# ── guided_choice ─────────────────────────────────────────
# La sortie sera EXACTEMENT l'une des valeurs
réponse = client.chat.completions.create(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    messages=[{"role": "user", "content": "Classifie : 'Mon colis est en retard.'"}],
    extra_body={"guided_choice": ["livraison", "retour", "paiement", "autre"]}
)
print(réponse.choices[0].message.content)   # → "livraison"

# ── guided_regex ──────────────────────────────────────────
# La sortie correspondra au pattern regex
réponse = client.chat.completions.create(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    messages=[{"role": "user", "content": "Date de la Révolution Française ?"}],
    extra_body={"guided_regex": r"(0[1-9]|[12][0-9]|3[01])/(0[1-9]|1[012])/[12][0-9]{3}"}
)
print(réponse.choices[0].message.content)   # → "14/07/1789"

# ── guided_json ───────────────────────────────────────────
# La sortie sera un JSON conforme au schéma
class AnalyseTicket(BaseModel):
    catégorie: Literal["livraison", "retour", "paiement", "autre"]
    priorité:  Literal["urgente", "haute", "normale", "faible"]
    résumé:    str
    actions:   List[str]

réponse = client.chat.completions.create(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    messages=[{
        "role": "user",
        "content": "Analyse : 'Colis endommagé à la livraison, je veux un échange.'"
    }],
    extra_body={"guided_json": AnalyseTicket.model_json_schema()}
)

texte = réponse.choices[0].message.content
résultat = AnalyseTicket.model_validate_json(texte)
print(résultat.catégorie)   # → "retour"
print(résultat.actions)     # → ["Proposer échange", "Vérifier stock"]
```

## Choisir le backend

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="no-key")

# XGrammar — production (100× plus rapide, recommandé)
réponse = client.chat.completions.create(
    model="...",
    messages=[...],
    extra_body={
        "guided_json": mon_schéma,
        "guided_decoding_backend": "xgrammar"
    }
)

# Outlines — flexible (FSM, support CFG complet)
réponse = client.chat.completions.create(
    model="...",
    messages=[...],
    extra_body={
        "guided_json": mon_schéma,
        "guided_decoding_backend": "outlines"
    }
)

# LM Format Enforcer — contraintes dynamiques
réponse = client.chat.completions.create(
    model="...",
    messages=[...],
    extra_body={
        "guided_json": mon_schéma,
        "guided_decoding_backend": "lm-format-enforcer"
    }
)
```

## guided_grammar — grammaire CFG custom

```python
grammaire_sql = r"""
    start: select_stmt
    select_stmt: "SELECT" columns "FROM" table ("WHERE" condition)?
    columns: "*" | column ("," column)*
    column: /[a-zA-Z_][a-zA-Z0-9_]*/
    table: /[a-zA-Z_][a-zA-Z0-9_]*/
    condition: column "=" value
    value: "'" /[^']*/ "'" | /[0-9]+/
    %ignore /\s+/
"""

réponse = client.chat.completions.create(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    messages=[{"role": "user", "content": "Génère une requête SQL pour les clients actifs"}],
    extra_body={"guided_grammar": grammaire_sql}
)
print(réponse.choices[0].message.content)
# → "SELECT nom, email FROM clients WHERE statut = 'actif'"
```

## API REST directe (sans client Python)

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistralai/Mistral-7B-Instruct-v0.3",
    "messages": [{"role": "user", "content": "Analyse ce ticket : retard de livraison"}],
    "extra_body": {
      "guided_json": {
        "type": "object",
        "properties": {
          "catégorie": {"type": "string", "enum": ["livraison", "retour", "autre"]},
          "priorité": {"type": "string", "enum": ["urgente", "haute", "normale"]},
          "résumé": {"type": "string"}
        },
        "required": ["catégorie", "priorité", "résumé"]
      }
    }
  }'
```

## Pipeline production complet

```python
from openai import OpenAI
from pydantic import BaseModel
from typing import Literal, List
import json

client = OpenAI(base_url="http://localhost:8000/v1", api_key="no-key")

class AnalyseTicket(BaseModel):
    catégorie:  Literal["livraison", "retour", "paiement", "garantie", "autre"]
    priorité:   Literal["urgente", "haute", "normale", "faible"]
    sentiment:  Literal["positif", "négatif", "neutre"]
    résumé:     str
    actions:    List[str]
    escalader:  bool

SCHÉMA = AnalyseTicket.model_json_schema()

def analyser_ticket(texte: str) -> AnalyseTicket:
    """Analyse un ticket avec garantie de sortie structurée."""
    réponse = client.chat.completions.create(
        model="mistralai/Mistral-7B-Instruct-v0.3",
        messages=[
            {"role": "system", "content": "Tu es un expert en analyse de tickets support."},
            {"role": "user",   "content": f"Analyse ce ticket : '{texte}'"}
        ],
        temperature=0.1,
        max_tokens=300,
        extra_body={
            "guided_json": SCHÉMA,
            "guided_decoding_backend": "xgrammar"
        }
    )

    texte_json = réponse.choices[0].message.content
    return AnalyseTicket.model_validate_json(texte_json)

# Test
résultat = analyser_ticket("URGENT : client VIP. Colis perdu depuis 3 semaines !")
print(f"Catégorie : {résultat.catégorie}")
print(f"Priorité  : {résultat.priorité}")
print(f"Escalader : {résultat.escalader}")
print(f"Actions   : {résultat.actions}")
```

> [!tip] XGrammar par défaut en production
> Lance vLLM avec `--guided-decoding-backend xgrammar` pour toutes les requêtes sans devoir le spécifier à chaque appel. XGrammar est le backend le plus rapide pour les formats JSON standards.

> [!warning] guided_grammar nécessite Outlines
> Le paramètre `guided_grammar` (grammaires CFG custom) n'est supporté que par le backend Outlines. XGrammar et LM Format Enforcer ont un support partiel. Si tu as besoin de grammaires CFG complexes, spécifie explicitement `"guided_decoding_backend": "outlines"`.
