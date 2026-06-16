#ia #transformers #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier torch.no_grad() en inférence

```python
# ❌ Sans no_grad → PyTorch garde tous les gradients en VRAM
outputs = model(**inputs)
# → VRAM 2-3× plus élevée que nécessaire, inférence plus lente

# ✅ Toujours no_grad en inférence
with torch.no_grad():
    outputs = model(**inputs)
    generated = model.generate(**inputs, max_new_tokens=200)
```

---

## 🪤 Piège 2 — return_full_text=True dans pipeline()

```python
# ❌ return_full_text=True (défaut) inclut le prompt dans la sortie
pipe = pipeline("text-generation", model="mistralai/Mistral-7B-Instruct-v0.3")
résultat = pipe("Qu'est-ce que le RAG ?", max_new_tokens=200)
print(résultat[0]["generated_text"])
# → "Qu'est-ce que le RAG ? Le RAG est une technique..." ← prompt inclus !

# ✅ Désactiver pour obtenir seulement la génération
résultat = pipe("Qu'est-ce que le RAG ?", max_new_tokens=200, return_full_text=False)
print(résultat[0]["generated_text"])
# → "Le RAG est une technique..." ← seulement la réponse
```

---

## 🪤 Piège 3 — do_sample=False + temperature

```python
# ❌ Temperature n'a aucun effet sans do_sample=True
outputs = model.generate(
    **inputs,
    temperature=0.7,    # ← ignoré !
    do_sample=False     # ← greedy : toujours le token le plus probable
)

# ✅ Activer do_sample pour que temperature soit prise en compte
outputs = model.generate(
    **inputs,
    temperature=0.7,
    do_sample=True,     # ← maintenant temperature est utilisée
    top_p=0.9
)
```

---

## 🪤 Piège 4 — Pad token manquant pour les batches

```python
# ❌ Sans pad token → erreur ou comportement incorrect sur les batches
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")
# tokenizer.pad_token est None pour Mistral

inputs = tokenizer(["texte 1", "texte plus long"], padding=True, return_tensors="pt")
# → Warning ou erreur : "tokenizer.pad_token is None"

# ✅ Définir le pad token avant les opérations de batch
tokenizer.pad_token = tokenizer.eos_token   # convention standard pour les LLM
# Ou
tokenizer.add_special_tokens({"pad_token": "[PAD]"})
model.resize_token_embeddings(len(tokenizer))
```

---

## 🪤 Piège 5 — Modèle sur CPU par inadvertance

```python
# ❌ Inputs sur GPU mais modèle sur CPU (ou inversement)
model = AutoModelForCausalLM.from_pretrained(model_id, device_map="auto")
inputs = tokenizer(texte, return_tensors="pt")   # inputs sur CPU par défaut !

with torch.no_grad():
    outputs = model(**inputs)   # → RuntimeError: Expected all tensors on same device

# ✅ Déplacer les inputs sur le même device que le modèle
inputs = tokenizer(texte, return_tensors="pt").to(model.device)
# Ou
inputs = {k: v.to(model.device) for k, v in inputs.items()}
```

---

## 🪤 Piège 6 — BitsAndBytes sur macOS ou CPU

```python
# ❌ BitsAndBytes nécessite CUDA (GPU NVIDIA sur Linux)
from transformers import BitsAndBytesConfig
bnb_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(model_id, quantization_config=bnb_config)
# → Sur macOS : ImportError ou RuntimeError

# ✅ Sur macOS (Apple Silicon) : utiliser mlx ou llama.cpp
# pip install mlx-lm
from mlx_lm import load, generate
model, tokenizer = load("mlx-community/Mistral-7B-Instruct-v0.3-4bit")
réponse = generate(model, tokenizer, prompt="Bonjour !")
```

---

## 🪤 Piège 7 — Mauvais apply_chat_template

```python
# ❌ Passer le texte brut à un modèle instruction-tuned
inputs = tokenizer("Qu'est-ce que le RAG ?", return_tensors="pt")
# → Le modèle ne "sait" pas que c'est une question utilisateur → réponse incohérente

# ✅ Utiliser le chat template du modèle
messages = [{"role": "user", "content": "Qu'est-ce que le RAG ?"}]
prompt = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True   # obligatoire pour la génération
)
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
```

---

## 🪤 Piège 8 — Oublier prepare_model_for_kbit_training avec QLoRA

```python
from peft import get_peft_model, LoraConfig
from peft import prepare_model_for_kbit_training

# ❌ Appliquer LoRA directement sur un modèle quantifié
model_4bit = AutoModelForCausalLM.from_pretrained(model_id, quantization_config=bnb_config)
model_lora = get_peft_model(model_4bit, lora_config)   # → erreur ou résultats dégradés

# ✅ Toujours préparer le modèle quantifié avant d'appliquer LoRA
model_4bit = prepare_model_for_kbit_training(model_4bit)   # ← étape obligatoire
model_lora = get_peft_model(model_4bit, lora_config)
```

---

## 🪤 Piège 9 — Flash Attention 2 sur GPU non compatible

```python
# ❌ Flash Attention 2 nécessite GPU Ampere+ (RTX 3000+, A10, A100...)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    attn_implementation="flash_attention_2"   # → erreur sur GPU plus ancien
)

# ✅ Vérifier la compatibilité
import torch
capability = torch.cuda.get_device_capability()
if capability[0] >= 8:   # SM80+ = Ampere+
    attn_impl = "flash_attention_2"
else:
    attn_impl = "sdpa"   # Scaled Dot Product Attention (PyTorch natif, compatible)

model = AutoModelForCausalLM.from_pretrained(model_id, attn_implementation=attn_impl)
```

---

## 🪤 Piège 10 — Cache HuggingFace qui manque d'espace disque

```python
# ❌ Le cache par défaut (~/.cache/huggingface) peut vite saturer
# Mistral 7B ≈ 14GB, LLaMA 70B ≈ 140GB

# ✅ Changer le répertoire de cache
import os
os.environ["HF_HOME"] = "/mon/disque/avec/de/lespace/hf_cache"
# Avant tout import de transformers !

# Ou dans le code
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    cache_dir="/mon/disque/hf_cache"
)

# Voir et nettoyer le cache
# huggingface-cli cache scan
# huggingface-cli cache delete
```

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Pas de torch.no_grad() | `with torch.no_grad():` en inférence |
| return_full_text=True | `return_full_text=False` dans pipeline() |
| Temperature sans do_sample | `do_sample=True` |
| Pad token manquant | `tokenizer.pad_token = tokenizer.eos_token` |
| Device mismatch | `.to(model.device)` sur les inputs |
| BnB sur macOS | Utiliser mlx ou llama.cpp |
| Pas de chat template | `tokenizer.apply_chat_template()` |
| QLoRA sans preparation | `prepare_model_for_kbit_training()` avant LoRA |
| FA2 sur vieux GPU | Vérifier SM80+, fallback sur `"sdpa"` |
| Cache plein | `HF_HOME` ou `cache_dir` personnalisé |
