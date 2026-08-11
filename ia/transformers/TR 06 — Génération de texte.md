#ia #transformers #génération #sampling #intermédiaire

## Génération de texte

La génération de texte consiste à prédire les tokens suivants un par un. Les paramètres de génération contrôlent la qualité, la créativité et la diversité des sorties.

## Les stratégies de décodage

```
Greedy decoding    : choisit toujours le token le plus probable
                     → Rapide, mais répétitif et peu créatif

Beam search        : explore plusieurs chemins simultanément
                     → Meilleur pour la traduction et le résumé

Sampling           : choisit aléatoirement selon les probabilités
                     → Créatif, mais peut diverger

Top-k sampling     : échantillonne parmi les k tokens les plus probables
Top-p (nucleus)    : échantillonne dans le noyau de probabilité cumulée p
Temperature        : contrôle la "chaleur" de la distribution
```

## Génération complète avec GenerationConfig

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, GenerationConfig
import torch

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForCausalLM.from_pretrained(
    modèle_id, torch_dtype=torch.float16, device_map="auto"
)

# Préparer le prompt avec le chat template
messages = [{"role": "user", "content": "Explique le RAG en 3 points."}]
prompt   = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs   = tokenizer(prompt, return_tensors="pt").to(model.device)

# ── Greedy (déterministe) ─────────────────────────────────
with torch.no_grad():
    outputs = model.generate(**inputs, max_new_tokens=200, do_sample=False)

# ── Temperature sampling (créatif) ────────────────────────
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=200,
        do_sample=True,
        temperature=0.7,        # 0.1=très déterministe, 2.0=très aléatoire
        top_k=50,               # garder seulement les 50 tokens les plus probables
        top_p=0.9,              # nucleus sampling : probabilité cumulée de 90%
        repetition_penalty=1.1  # pénaliser les répétitions (1.0=aucune pénalité)
    )

# ── Beam search (qualité maximale) ────────────────────────
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=200,
        num_beams=4,            # explorer 4 chemins simultanément
        no_repeat_ngram_size=3, # pas de trigramme répété
        early_stopping=True     # s'arrêter si tous les beams ont fini
    )

# ── Décoder la sortie ─────────────────────────────────────
tokens_nouveaux = outputs[0][inputs["input_ids"].shape[1]:]
réponse = tokenizer.decode(tokens_nouveaux, skip_special_tokens=True)
print(réponse)
```

## Streaming — afficher token par token

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TextStreamer
import torch

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForCausalLM.from_pretrained(
    modèle_id, torch_dtype=torch.float16, device_map="auto"
)

messages = [{"role": "user", "content": "Explique le RAG."}]
prompt   = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs   = tokenizer(prompt, return_tensors="pt").to(model.device)

# TextStreamer affiche chaque token dès qu'il est généré
streamer = TextStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)

with torch.no_grad():
    model.generate(
        **inputs,
        max_new_tokens=300,
        do_sample=True,
        temperature=0.7,
        streamer=streamer   # ← active le streaming
    )
# → Les mots s'affichent progressivement, comme ChatGPT
```

## Streaming asynchrone pour les APIs

```python
from transformers import TextIteratorStreamer
from threading import Thread

streamer = TextIteratorStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)

# Lancer la génération dans un thread séparé
thread = Thread(target=model.generate, kwargs={
    **inputs,
    "max_new_tokens": 300,
    "do_sample": True,
    "streamer": streamer
})
thread.start()

# Consommer les tokens au fur et à mesure (pour FastAPI par ex.)
réponse_complète = ""
for chunk in streamer:
    réponse_complète += chunk
    print(chunk, end="", flush=True)   # ou yield chunk dans une API

thread.join()
```

## Paramètres de génération — guide pratique

| Paramètre | Valeur | Effet |
|---|---|---|
| `temperature=0.1` | Très bas | Déterministe, répétitif |
| `temperature=0.7` | Modéré | Bon équilibre (défaut recommandé) |
| `temperature=1.5` | Élevé | Très créatif, parfois incohérent |
| `top_k=50` | Standard | Filtre les 50 tokens les plus probables |
| `top_p=0.9` | Standard | Nucleus sampling à 90% |
| `top_p=1.0` | Désactivé | Pas de nucleus sampling |
| `repetition_penalty=1.1` | Léger | Évite les répétitions légères |
| `repetition_penalty=1.5` | Fort | Évite fortement les répétitions |
| `num_beams=4` | Beam search | Plus qualitatif, plus lent |
| `do_sample=False` | Greedy | 100% déterministe |

## Contrôle des tokens d'arrêt

```python
# Arrêter la génération sur des séquences spécifiques
from transformers import StoppingCriteria, StoppingCriteriaList

class StopOnSequence(StoppingCriteria):
    def __init__(self, stop_ids: list):
        self.stop_ids = stop_ids

    def __call__(self, input_ids, scores, **kwargs) -> bool:
        for stop_id in self.stop_ids:
            if input_ids[0, -len(stop_id):].tolist() == stop_id:
                return True
        return False

# Arrêter sur "---" ou sur "Fin."
stop_ids = [
    tokenizer.encode("---", add_special_tokens=False),
    tokenizer.encode("Fin.", add_special_tokens=False)
]

stopping_criteria = StoppingCriteriaList([StopOnSequence(stop_ids)])

with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=500,
        stopping_criteria=stopping_criteria
    )
```

## GenerationConfig — configuration réutilisable

```python
from transformers import GenerationConfig

# Créer une config réutilisable
config_créatif = GenerationConfig(
    max_new_tokens=500,
    do_sample=True,
    temperature=0.8,
    top_p=0.9,
    repetition_penalty=1.1
)

config_précis = GenerationConfig(
    max_new_tokens=200,
    do_sample=False,   # greedy
    num_beams=4
)

# Utiliser
with torch.no_grad():
    outputs = model.generate(**inputs, generation_config=config_créatif)

# Sauvegarder / charger
config_créatif.save_pretrained("./ma_config")
config_chargée = GenerationConfig.from_pretrained("./ma_config")
```

> [!tip] Combinaison recommandée pour le RAG
> Pour une application RAG où la précision prime : `temperature=0.3, top_p=0.9, do_sample=True, repetition_penalty=1.1`. Assez déterministe pour être fiable, assez flexible pour éviter les répétitions.

> [!warning] do_sample=False + temperature ≠ logique
> Si `do_sample=False` (greedy), la `temperature` n'a aucun effet. Pour que la temperature soit prise en compte, il faut `do_sample=True`.
