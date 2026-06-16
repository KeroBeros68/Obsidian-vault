#ia #constrained-decoding #pièges #erreurs #debugging

## 🪤 Piège 1 — Recréer le générateur à chaque requête

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# ❌ Compiler à chaque requête → startup cost répété (2-5s par requête !)
def analyser(ticket: str):
    générateur = outlines.generate.json(model, AnalyseTicket)   # ← compilation !
    return générateur(f"Analyse : '{ticket}'")

# ✅ Compiler une fois au démarrage, réutiliser
générateur = outlines.generate.json(model, AnalyseTicket)   # ← une seule fois

def analyser(ticket: str):
    return générateur(f"Analyse : '{ticket}'")   # ← quasi-instantané
```

---

## 🪤 Piège 2 — Oublier le chat template avec Outlines

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")
générateur = outlines.generate.json(model, AnalyseTicket)

# ❌ Passer le texte brut à un modèle instruct
résultat = générateur("Analyse ce ticket : 'Colis en retard.'")
# → Mauvaise réponse : le modèle instruct attend un format spécifique

# ✅ Appliquer le chat template manuellement
from transformers import AutoTokenizer
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")

messages = [{"role": "user", "content": "Analyse ce ticket : 'Colis en retard.'"}]
prompt   = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)

résultat = générateur(prompt)   # ← prompt formaté correctement
```

> [!warning] Outlines ne gère pas les chat templates automatiquement
> C'est le piège le plus courant avec Outlines. Contrairement à Transformers `pipeline()`, Outlines ne préprocesse pas le prompt. Toujours appliquer `apply_chat_template()` avant de passer à Outlines.

---

## 🪤 Piège 3 — Schéma Pydantic trop complexe / récursif

```python
from pydantic import BaseModel
from typing import List, Optional

# ❌ Schéma récursif — peut planter Outlines ou prendre très longtemps à compiler
class NœudArbre(BaseModel):
    valeur: str
    enfants: Optional[List["NœudArbre"]] = None   # ← récursif infini

générateur = outlines.generate.json(model, NœudArbre)
# → Compilation infinie ou crash mémoire

# ✅ Limiter la profondeur explicitement
class NœudNiveau3(BaseModel):
    valeur: str

class NœudNiveau2(BaseModel):
    valeur: str
    enfants: Optional[List[NœudNiveau3]] = None

class NœudRacine(BaseModel):
    valeur: str
    enfants: Optional[List[NœudNiveau2]] = None
```

---

## 🪤 Piège 4 — guided_grammar avec un backend incompatible

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="no-key")

grammaire_sql = "start: ..."

# ❌ guided_grammar avec xgrammar (support partiel)
réponse = client.chat.completions.create(
    model="...",
    messages=[...],
    extra_body={
        "guided_grammar": grammaire_sql,
        "guided_decoding_backend": "xgrammar"   # ← peut ne pas supporter CFG complexe
    }
)

# ✅ Pour les grammaires CFG, utiliser Outlines explicitement
réponse = client.chat.completions.create(
    model="...",
    messages=[...],
    extra_body={
        "guided_grammar": grammaire_sql,
        "guided_decoding_backend": "outlines"   # ← support CFG complet
    }
)
```

---

## 🪤 Piège 5 — Contrainte trop restrictive qui dégrade la qualité

```python
# ❌ Regex trop stricte qui force des résultats absurdes
import outlines

générateur = outlines.generate.regex(
    model,
    r"[A-Z][a-z]+ [A-Z][a-z]+"   # Format "Prénom Nom" très strict
)

# Prompt : "Quel est le nom de la tour Eiffel ?"
résultat = générateur("Quel est le nom de la tour Eiffel ?")
# → "Tour Eiffel" ← forcé par la regex, même si la réponse correcte est "la tour Eiffel"

# ✅ Contrainte plus flexible
générateur = outlines.generate.regex(
    model,
    r"[A-Za-zÀ-ÿ\s\-']+"   # texte avec accents, espaces, tirets
)
```

---

## 🪤 Piège 6 — Ne pas valider après génération malgré la contrainte

```python
# ❌ Même avec constrained decoding, un parsing sans validation peut masquer des bugs
import json

texte_json = résultat_généré
data = json.loads(texte_json)   # peut réussir même avec un schéma partiel
print(data["catégorie"])        # pas de validation de type

# ✅ Toujours valider avec Pydantic après génération
from pydantic import BaseModel, ValidationError

try:
    objet = MonSchéma.model_validate_json(texte_json)
    print(objet.catégorie)   # ← validé et typé
except ValidationError as e:
    print(f"Validation échouée malgré la contrainte : {e}")
    # → Cas très rare mais possible (bug dans la librairie, schéma mal formé)
```

---

## 🪤 Piège 7 — Utiliser Outlines avec une API cloud (impossible)

```python
# ❌ Outlines modèle local ≠ API cloud
import outlines

# Outlines transformers fonctionne UNIQUEMENT avec des modèles locaux
model = outlines.models.transformers("mistralai/Mistral-7B-Instruct-v0.3")

# Pour les APIs cloud (Claude, GPT-4), utiliser :
# → with_structured_output() de LangChain
# → response_format (OpenAI)
# → Outlines ne peut pas intercepter les tokens d'une API fermée

# Pour Claude
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-sonnet-4-20250514")
llm_structuré = llm.with_structured_output(MonSchéma)
```

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Recréer le générateur à chaque requête | Compiler une fois au démarrage, réutiliser |
| Oublier le chat template (Outlines) | `tokenizer.apply_chat_template()` avant Outlines |
| Schéma Pydantic récursif | Limiter la profondeur d'imbrication explicitement |
| guided_grammar + xgrammar | Utiliser `"outlines"` pour les grammaires CFG |
| Contrainte trop restrictive | Tester la regex/schéma sur des cas limites avant prod |
| Pas de validation Pydantic après génération | Toujours `MonSchéma.model_validate_json()` |
| Outlines avec API cloud | Impossible — utiliser `with_structured_output()` |
