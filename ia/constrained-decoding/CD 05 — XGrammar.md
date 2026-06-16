#ia #constrained-decoding #xgrammar #performance #production

## XGrammar

XGrammar est un moteur de génération structurée haute performance qui accélère les LLM de **100×** grâce à des masques de tokens pré-calculés et du traitement parallèle. C'est le backend recommandé quand la performance est critique.

## Pourquoi XGrammar ?

XGrammar est un moteur haute performance qui accélère les LLM de 100× en utilisant des masques de tokens pré-calculés et du traitement parallèle. Il est particulièrement adapté aux structures complexes nécessitant l'application de grammaires context-free avec de hautes exigences de performance.

```
Outlines         : pré-calcule les masques pour chaque état FSM
                   → startup coûteux, inférence rapide

LM Format Enforcer : calcule les masques dynamiquement
                   → startup nul, inférence moins rapide

XGrammar         : masques pré-calculés + traitement parallèle CPU/GPU
                   → le meilleur des deux mondes en production
                   → 100× plus rapide que LM Format Enforcer
                   → support CFG complet
```

## Installation et usage

```bash
pip install xgrammar
```

```python
import xgrammar as xgr
from transformers import AutoTokenizer

# Initialiser XGrammar avec le tokenizer
tokenizer  = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")
tokenizer_info = xgr.TokenizerInfo.from_huggingface(tokenizer)
compiler   = xgr.GrammarCompiler(tokenizer_info)

# Compiler une contrainte JSON Schema
import json
from pydantic import BaseModel
from typing import Literal

class AnalyseTicket(BaseModel):
    catégorie: Literal["livraison", "retour", "paiement", "autre"]
    priorité:  Literal["urgente", "haute", "normale", "faible"]
    résumé:    str

compiled_grammar = compiler.compile_json_schema(
    json.dumps(AnalyseTicket.model_json_schema())
)

# Créer le logits processor
logits_processor = xgr.contrib.hf.LogitsProcessor(compiled_grammar)

# Utiliser avec Transformers
from transformers import AutoModelForCausalLM
import torch

model  = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-Instruct-v0.3",
    torch_dtype=torch.float16,
    device_map="auto"
)

inputs = tokenizer(
    "Analyse ce ticket : 'Mon colis est en retard depuis 10 jours.'",
    return_tensors="pt"
).to(model.device)

with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=200,
        logits_processor=[logits_processor]   # ← XGrammar
    )

texte = tokenizer.decode(outputs[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True)
résultat = AnalyseTicket.model_validate_json(texte)
print(résultat.catégorie)   # → "livraison"
```

## XGrammar avec vLLM

XGrammar est disponible comme backend guided decoding dans vLLM.

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="no-key")

réponse = client.chat.completions.create(
    model="mistralai/Mistral-7B-Instruct-v0.3",
    messages=[{"role": "user", "content": "Analyse ce ticket : 'Retard de livraison.'"}],
    extra_body={
        "guided_json": AnalyseTicket.model_json_schema(),
        "guided_decoding_backend": "xgrammar"   # ← spécifier XGrammar
    }
)
print(réponse.choices[0].message.content)
```

## Grammaires CFG avec XGrammar

```python
import xgrammar as xgr

# Grammaire EBNF pour du JSON simplifié
grammaire_ebnf = r"""
root ::= object
object ::= "{" pair ("," pair)* "}"
pair ::= string ":" value
value ::= string | number | "true" | "false" | "null"
string ::= "\"" [^"]* "\""
number ::= "-"? [0-9]+ ("." [0-9]+)?
"""

compiled = compiler.compile_grammar(grammaire_ebnf)
logits_processor = xgr.contrib.hf.LogitsProcessor(compiled)
```

> [!tip] XGrammar en production
> XGrammar est le choix production quand tu as un grand volume de requêtes et un format fixe. Sa compilation une fois au démarrage + traitement parallèle le rend idéal pour les APIs à fort trafic.
