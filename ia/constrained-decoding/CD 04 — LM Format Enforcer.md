#ia #constrained-decoding #lm-format-enforcer #pratique

## LM Format Enforcer

LM Format Enforcer assure la conformité au format via un **filtrage token par token dynamique**. Plus flexible qu'Outlines pour les contraintes dynamiques ou paramétrées, et particulièrement efficace pour réduire les hallucinations de format.

## Installation

```bash
pip install lm-format-enforcer
```

## Principe — filtrage dynamique

```
Outlines       : pré-compile toute la FSM → lookup O(1) mais startup cost
LM Format Enforcer : calcule les tokens valides dynamiquement à chaque step
                    → pas de startup cost, plus flexible
                    → légèrement plus lent sur les formats statiques
                    → meilleur quand le format dépend des données en entrée
```

## Avec Transformers — JSON Schema

```python
from lmformatenforcer import JsonSchemaParser
from lmformatenforcer.integrations.transformers import (
    build_transformers_prefix_allowed_tokens_fn
)
from transformers import AutoModelForCausalLM, AutoTokenizer
from pydantic import BaseModel
from typing import Literal, List
import torch

# Charger le modèle
modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForCausalLM.from_pretrained(
    modèle_id, torch_dtype=torch.float16, device_map="auto"
)

# Définir le schéma
class AnalyseTicket(BaseModel):
    catégorie:  Literal["livraison", "retour", "paiement", "autre"]
    priorité:   Literal["urgente", "haute", "normale", "faible"]
    résumé:     str
    confiance:  float

# Créer le parser LM Format Enforcer
parser = JsonSchemaParser(AnalyseTicket.model_json_schema())

# Créer la fonction de filtrage (prefix_allowed_tokens_fn)
prefix_fn = build_transformers_prefix_allowed_tokens_fn(tokenizer, parser)

# Préparer le prompt
messages = [{"role": "user", "content": "Analyse ce ticket : 'Mon colis est en retard.'"}]
prompt   = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs   = tokenizer(prompt, return_tensors="pt").to(model.device)

# Générer avec contrainte
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=200,
        prefix_allowed_tokens_fn=prefix_fn   # ← la magie est ici
    )

# Décoder
texte_généré = tokenizer.decode(
    outputs[0][inputs["input_ids"].shape[1]:],
    skip_special_tokens=True
)
print(texte_généré)
# → {"catégorie": "livraison", "priorité": "haute", "résumé": "...", "confiance": 0.92}

import json
résultat = AnalyseTicket(**json.loads(texte_généré))
print(résultat.catégorie)   # → "livraison"
```

## Avec Transformers — Regex

```python
from lmformatenforcer import RegexParser
from lmformatenforcer.integrations.transformers import (
    build_transformers_prefix_allowed_tokens_fn
)

# Parser regex
parser_date = RegexParser(r"(0[1-9]|[12][0-9]|3[01])/(0[1-9]|1[012])/[12][0-9]{3}")
prefix_fn   = build_transformers_prefix_allowed_tokens_fn(tokenizer, parser_date)

inputs = tokenizer("Date de la Révolution Française :", return_tensors="pt").to(model.device)
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=10,
        prefix_allowed_tokens_fn=prefix_fn
    )

date = tokenizer.decode(outputs[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True)
print(date)   # → "14/07/1789"
```

## Avec vLLM

```python
from vllm import LLM, SamplingParams
from lmformatenforcer import JsonSchemaParser
from lmformatenforcer.integrations.vllm import (
    build_vllm_logits_processor
)

# Charger le modèle avec vLLM
llm = LLM(model="mistralai/Mistral-7B-Instruct-v0.3")

# Créer le parser et le logits processor
parser    = JsonSchemaParser(AnalyseTicket.model_json_schema())
processor = build_vllm_logits_processor(llm, parser)

# Paramètres de sampling avec le processor
params = SamplingParams(
    temperature=0.1,
    max_tokens=200,
    logits_processors=[processor]
)

# Générer
outputs = llm.generate(
    ["Analyse ce ticket : 'Remboursement refusé.'"],
    sampling_params=params
)
print(outputs[0].outputs[0].text)
```

## LM Format Enforcer avec contraintes dynamiques

L'avantage clé : les contraintes peuvent dépendre du contexte ou de données en entrée.

```python
from lmformatenforcer import JsonSchemaParser
from pydantic import BaseModel, create_model
from typing import Literal

def créer_parser_dynamique(catégories_valides: list[str]) -> JsonSchemaParser:
    """Crée un parser dont les catégories valides dépendent du contexte."""

    # Créer un modèle Pydantic dynamiquement
    ModèleDynamique = create_model(
        "ModèleDynamique",
        catégorie=(Literal[tuple(catégories_valides)], ...),
        confiance=(float, ...),
        résumé=(str, ...)
    )

    return JsonSchemaParser(ModèleDynamique.model_json_schema())

# Pour un client VIP → catégories différentes
parser_vip = créer_parser_dynamique(["remboursement_prioritaire", "escalade_manager", "autre"])

# Pour un client standard
parser_std = créer_parser_dynamique(["livraison", "retour", "paiement", "autre"])
```

## Hallucinations réduites : chiffres clés

D'après l'étude comparative 2025 (Outlines vs XGrammar vs LM Format Enforcer sur des pipelines RAG) :

- En zero-shot, LM Format Enforcer réduit significativement les hallucinations à seulement 8,9%, prouvant son efficacité pour l'application de formats stricts.
- En one-shot, Outlines atteint le taux d'hallucination le plus bas à 1,8%, surpassant les autres backends. En two-shot, Outlines atteint 0,4% d'hallucination, quasiment éliminant les erreurs, tandis que LM Format Enforcer performe également très bien à 0,7%.

## Comparaison Outlines vs LM Format Enforcer

| Critère | Outlines | LM Format Enforcer |
|---|---|---|
| **Mécanisme** | FSM pré-compilée | Filtrage dynamique |
| **Startup cost** | Quelques secondes | Quasi nul |
| **Vitesse inférence** | Très rapide (O(1)) | Moins rapide |
| **Contraintes dynamiques** | Limitées | ✅ Excellentes |
| **Support regex** | ✅ | ✅ |
| **Support JSON** | ✅ | ✅ |
| **Support CFG** | ✅ | Partiel |
| **Intégration vLLM** | Native | Via logits processor |

> [!tip] LM Format Enforcer pour les formats paramétrés
> Si ton format dépend de données en entrée (catégories variables, schéma généré à la volée), LM Format Enforcer est bien plus adapté qu'Outlines. Si ton format est fixe, Outlines sera plus rapide en inférence.
