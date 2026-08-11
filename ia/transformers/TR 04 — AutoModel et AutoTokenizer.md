#ia #transformers #automodel #tokenizer #bases

## AutoModel et AutoTokenizer

Les classes `Auto*` détectent automatiquement l'architecture du modèle et chargent la bonne classe. C'est l'API de bas niveau — plus de contrôle que `pipeline()`.

## AutoTokenizer — transformer du texte en tokens

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")

# ── Encoder du texte en tokens ────────────────────────────
texte = "Bonjour, comment vas-tu ?"

# Tokenisation simple
tokens = tokenizer.tokenize(texte)
print(tokens)
# → ['▁Bon', 'jour', ',', '▁comment', '▁vas', '-', 'tu', '▁?']

# Conversion en IDs numériques
ids = tokenizer.convert_tokens_to_ids(tokens)
print(ids)   # → [661, 1785, 28725, 910, 3185, 28733, 10730, 1550]

# Encode complet (tokenise + convertit en tenseur)
encoding = tokenizer(texte, return_tensors="pt")
print(encoding["input_ids"])   # tenseur PyTorch
print(encoding["attention_mask"])

# ── Décoder des IDs en texte ──────────────────────────────
texte_décodé = tokenizer.decode(encoding["input_ids"][0])
print(texte_décodé)   # → "<s> Bonjour, comment vas-tu ?"

# Sans les tokens spéciaux
texte_propre = tokenizer.decode(
    encoding["input_ids"][0],
    skip_special_tokens=True
)
print(texte_propre)   # → "Bonjour, comment vas-tu ?"
```

## Tokens spéciaux

```python
# Voir les tokens spéciaux du modèle
print(tokenizer.bos_token)    # → "<s>"  (début de séquence)
print(tokenizer.eos_token)    # → "</s>" (fin de séquence)
print(tokenizer.pad_token)    # → None ou "<pad>"
print(tokenizer.unk_token)    # → "<unk>" (token inconnu)
print(tokenizer.sep_token)    # → utilisé pour séparer les segments (BERT)

# Ajouter un token de padding si absent (nécessaire pour les batches)
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token
```

## Batch de textes avec padding/truncation

```python
textes = [
    "Texte court.",
    "Texte beaucoup plus long qui nécessite un padding ou une troncature."
]

encoding = tokenizer(
    textes,
    padding=True,          # pad au même longueur
    truncation=True,       # tronquer si > max_length
    max_length=512,        # longueur maximale
    return_tensors="pt"    # tenseurs PyTorch
)

print(encoding["input_ids"].shape)      # → (2, 512)
print(encoding["attention_mask"].shape) # → (2, 512)
# attention_mask : 1=vrai token, 0=padding
```

## Appliquer le chat template

Pour les modèles instruction-tuned, le prompt doit suivre un format précis.

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")

# Format de conversation
messages = [
    {"role": "system", "content": "Tu es un assistant utile."},
    {"role": "user",   "content": "Explique le RAG en 2 phrases."},
]

# Appliquer le template du modèle
prompt = tokenizer.apply_chat_template(
    messages,
    tokenize=False,         # False = retourne la string formatée
    add_generation_prompt=True  # ajoute le token de début de génération
)
print(prompt)
# → "<s>[INST] Tu es un assistant utile.\n\nExplique le RAG en 2 phrases. [/INST]"

# Tokeniser le prompt formaté
inputs = tokenizer(prompt, return_tensors="pt")
```

## AutoModel — charger le modèle

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"

tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    torch_dtype=torch.float16,   # float16 = 2× moins de VRAM que float32
    device_map="auto"            # distribue sur les GPU disponibles
)

# Voir la taille du modèle
nb_params = sum(p.numel() for p in model.parameters())
print(f"Paramètres : {nb_params / 1e9:.1f}B")   # → 7.2B
```

## Les classes Auto* selon la tâche

```python
from transformers import (
    AutoModel,                    # modèle de base (features)
    AutoModelForCausalLM,         # génération de texte (GPT, Mistral, LLaMA)
    AutoModelForSeq2SeqLM,        # seq2seq (BART, T5, traduction, résumé)
    AutoModelForSequenceClassification,  # classification de texte
    AutoModelForTokenClassification,     # NER, POS tagging
    AutoModelForQuestionAnswering,       # QA extractive
    AutoModelForMaskedLM,               # BERT, RoBERTa (fill-mask)
    AutoModelForSpeechSeq2Seq,          # Whisper (ASR)
)
```

## Inférence complète — tokenizer + model

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForCausalLM.from_pretrained(
    modèle_id, torch_dtype=torch.float16, device_map="auto"
)

# Préparer le prompt
messages = [{"role": "user", "content": "Qu'est-ce que le RAG ?"}]
prompt   = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs   = tokenizer(prompt, return_tensors="pt").to(model.device)

# Générer
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=200,
        temperature=0.7,
        do_sample=True,
        pad_token_id=tokenizer.eos_token_id
    )

# Décoder — seulement la partie générée (après le prompt)
tokens_générés = outputs[0][inputs["input_ids"].shape[1]:]
réponse = tokenizer.decode(tokens_générés, skip_special_tokens=True)
print(réponse)
```

## Sauvegarder et charger un modèle local

```python
# Sauvegarder
model.save_pretrained("./mon_modele_sauvegardé")
tokenizer.save_pretrained("./mon_modele_sauvegardé")

# Charger depuis le disque (sans connexion internet)
model     = AutoModelForCausalLM.from_pretrained("./mon_modele_sauvegardé")
tokenizer = AutoTokenizer.from_pretrained("./mon_modele_sauvegardé")
```

> [!tip] device_map="auto" — le paramètre magique
> Avec `device_map="auto"`, Transformers distribue automatiquement le modèle sur tous les GPU disponibles, et met ce qui déborde en RAM CPU. Indispensable pour les grands modèles.

> [!tip] low_cpu_mem_usage=True — charger progressivement plutôt que doubler la RAM
> Sans cette option, `from_pretrained()` alloue d'abord le modèle complet en mémoire avant d'y charger les poids — un pic mémoire qui peut atteindre le double de la taille finale du modèle. `low_cpu_mem_usage=True` charge les poids progressivement, évitant ce pic, particulièrement utile en environnement contraint (CPU seul, RAM limitée).

> [!warning] torch.no_grad() en inférence
> Toujours utiliser `with torch.no_grad():` pendant l'inférence. Sans ça, PyTorch garde en mémoire tous les gradients intermédiaires — inutile en inférence et très coûteux en VRAM.
