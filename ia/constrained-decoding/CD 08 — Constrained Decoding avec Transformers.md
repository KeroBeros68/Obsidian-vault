#ia #constrained-decoding #transformers #logits-processor #pratique

## Constrained Decoding avec Transformers

Avec la librairie Transformers directement, le constrained decoding s'intègre via les `LogitsProcessor`. Outlines, LM Format Enforcer et XGrammar fournissent tous des wrappers pour Transformers.

## Via Outlines + Transformers

```python
import outlines
from pydantic import BaseModel
from typing import Literal, List

# Charger le modèle via Outlines
model = outlines.models.transformers(
    "mistralai/Mistral-7B-Instruct-v0.3",
    device="cuda",
    model_kwargs={"torch_dtype": "float16"}
)

class AnalyseTicket(BaseModel):
    catégorie: Literal["livraison", "retour", "paiement", "autre"]
    priorité:  Literal["urgente", "haute", "normale", "faible"]
    résumé:    str
    actions:   List[str]

générateur = outlines.generate.json(model, AnalyseTicket)

résultat = générateur(
    "Analyse ce ticket : 'Mon colis est arrivé cassé, je veux être remboursé.'"
)
print(résultat.catégorie)   # → "retour"
print(résultat.priorité)    # → "haute"
```

## Via XGrammar + Transformers (LogitsProcessor)

```python
import xgrammar as xgr
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from pydantic import BaseModel
from typing import Literal
import json

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForCausalLM.from_pretrained(
    modèle_id, torch_dtype=torch.float16, device_map="auto"
)

# Compiler la contrainte avec XGrammar
tokenizer_info = xgr.TokenizerInfo.from_huggingface(tokenizer)
compiler       = xgr.GrammarCompiler(tokenizer_info)

class Résultat(BaseModel):
    valeur:    float
    unité:     Literal["km", "m", "cm", "mm"]
    précision: int

compiled = compiler.compile_json_schema(json.dumps(Résultat.model_json_schema()))
processor = xgr.contrib.hf.LogitsProcessor(compiled)

# Générer avec XGrammar
messages = [{"role": "user", "content": "Distance Terre-Lune en km ?"}]
prompt   = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs   = tokenizer(prompt, return_tensors="pt").to(model.device)

with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=100,
        logits_processors=[processor]
    )

texte = tokenizer.decode(outputs[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True)
résultat = Résultat.model_validate_json(texte)
print(f"{résultat.valeur} {résultat.unité}")   # → "384400.0 km"
```

## Créer un LogitsProcessor custom

Pour des contraintes très spécifiques non couvertes par les librairies existantes.

```python
import torch
from transformers import LogitsProcessor, LogitsProcessorList

class ContreinteLongueurToken(LogitsProcessor):
    """Force la génération à s'arrêter après exactement N tokens."""

    def __init__(self, max_tokens: int, eos_token_id: int):
        self.max_tokens   = max_tokens
        self.eos_token_id = eos_token_id
        self.compteur     = 0

    def __call__(
        self,
        input_ids: torch.LongTensor,
        scores: torch.FloatTensor
    ) -> torch.FloatTensor:
        self.compteur += 1
        if self.compteur >= self.max_tokens:
            # Forcer EOS — masquer tous les autres tokens
            masque = torch.full_like(scores, float("-inf"))
            masque[:, self.eos_token_id] = 0
            return masque
        return scores

class ContreinteMots(LogitsProcessor):
    """Interdit des tokens spécifiques (liste noire)."""

    def __init__(self, tokens_interdits: list[int]):
        self.tokens_interdits = tokens_interdits

    def __call__(self, input_ids, scores):
        scores[:, self.tokens_interdits] = float("-inf")
        return scores

# Utilisation combinée
processor_list = LogitsProcessorList([
    ContreinteLongueurToken(max_tokens=50, eos_token_id=tokenizer.eos_token_id),
    ContreinteMots(tokens_interdits=[tokenizer.encode("malheureusement")[0]])
])

with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=100,
        logits_processor=processor_list
    )
```

## Pipeline d'extraction structurée complet

Cas réel : extraire des entités structurées depuis du texte libre.

```python
import outlines
import json
from pydantic import BaseModel, Field
from typing import Optional, List

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

class EntitésExtraites(BaseModel):
    personnes:    List[str] = Field(default_factory=list, description="Noms de personnes")
    organisations: List[str] = Field(default_factory=list, description="Noms d'organisations")
    lieux:        List[str] = Field(default_factory=list, description="Noms de lieux")
    dates:        List[str] = Field(default_factory=list, description="Dates mentionnées")
    montants:     List[str] = Field(default_factory=list, description="Montants en euros")

générateur = outlines.generate.json(model, EntitésExtraites)

texte = """
Emmanuel Macron a rencontré Tim Cook (Apple) à Paris le 15 mars 2025.
Le contrat signé est estimé à 2,5 milliards d'euros. La réunion s'est
tenue au Palais de l'Élysée avec des délégués de Microsoft et Google.
"""

résultat = générateur(
    f"Extrais toutes les entités nommées de ce texte :\n\n{texte}\n\nRéponds en JSON structuré."
)

print("Personnes:", résultat.personnes)        # → ["Emmanuel Macron", "Tim Cook"]
print("Organisations:", résultat.organisations) # → ["Apple", "Microsoft", "Google"]
print("Lieux:", résultat.lieux)               # → ["Paris", "Palais de l'Élysée"]
print("Dates:", résultat.dates)               # → ["15 mars 2025"]
print("Montants:", résultat.montants)         # → ["2,5 milliards d'euros"]
```

## Batch avec Transformers + Outlines

```python
import outlines
from pydantic import BaseModel
from typing import Literal

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

class Classification(BaseModel):
    catégorie: Literal["spam", "ham", "promotion", "social"]
    score: float

générateur = outlines.generate.json(model, Classification)

emails = [
    "Félicitations ! Vous avez gagné 1000€ ! Cliquez ici !",
    "Réunion demain à 14h en salle B.",
    "50% de réduction sur tous nos articles ce week-end !"
]

prompts = [f"Classifie cet email : '{e}'" for e in emails]
résultats = générateur(prompts)   # batch processing

for email, résultat in zip(emails, résultats):
    print(f"{résultat.catégorie:10} ({résultat.score:.2f}) | {email[:40]}")
```

> [!tip] Compiler le générateur une fois, réutiliser mille fois
> Avec Outlines ou XGrammar, la compilation (FSM ou masques) se fait à la création du générateur. Stocke le générateur comme attribut de classe ou variable globale — ne le recrée pas à chaque requête.
