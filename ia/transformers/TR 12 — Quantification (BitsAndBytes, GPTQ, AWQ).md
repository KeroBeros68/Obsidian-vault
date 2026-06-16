#ia #transformers #quantification #bitsandbytes #gptq #awq #optimisation #avancé

## Quantification — BitsAndBytes, GPTQ, AWQ

La quantification réduit la précision numérique des poids du modèle (float32 → int8 ou int4), réduisant drastiquement la VRAM nécessaire avec une perte de qualité minimale.

## Pourquoi quantifier ?

```
Mistral 7B en différentes précisions :
  float32  : 28 GB VRAM  (4 bytes × 7B params)
  float16  : 14 GB VRAM  (2 bytes × 7B params)
  int8     :  7 GB VRAM  (1 byte  × 7B params)
  int4     :  3.5 GB VRAM (0.5 byte × 7B params)
  
→ Un GPU RTX 3090 (24GB) peut faire tourner Mistral 7B en fp16
→ Un GPU RTX 3060 (12GB) peut faire tourner Mistral 7B en int4
```

## BitsAndBytes — quantification dynamique

Quantification à la volée au chargement. La plus simple à utiliser.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
import torch

# ── 8-bit (bonne qualité, moins d'économie) ───────────────
bnb_8bit = BitsAndBytesConfig(load_in_8bit=True)

model_8bit = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-Instruct-v0.3",
    quantization_config=bnb_8bit,
    device_map="auto"
)

# ── 4-bit NF4 (recommandé pour QLoRA) ─────────────────────
bnb_4bit = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",             # NormalFloat4 (meilleur que fp4)
    bnb_4bit_compute_dtype=torch.bfloat16, # calcul en bfloat16
    bnb_4bit_use_double_quant=True         # double quantification → -15% VRAM supplémentaire
)

model_4bit = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-Instruct-v0.3",
    quantization_config=bnb_4bit,
    device_map="auto"
)

# Utilisation identique en inférence
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")
inputs = tokenizer("Bonjour !", return_tensors="pt").to(model_4bit.device)
with torch.no_grad():
    outputs = model_4bit.generate(**inputs, max_new_tokens=100)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

## GPTQ — quantification post-entraînement précise

GPTQ calibre la quantification sur un dataset, donnant de meilleurs résultats que BitsAndBytes à faible précision. Les modèles GPTQ sont souvent disponibles pré-quantifiés sur le Hub.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from optimum.gptq import GPTQQuantizer, load_quantized_model
import torch

# ── Utiliser un modèle GPTQ pré-quantifié (recommandé) ────
# Chercher sur HuggingFace : "nom-du-modele GPTQ"
# Ex : TheBloke/Mistral-7B-Instruct-v0.2-GPTQ

model = AutoModelForCausalLM.from_pretrained(
    "TheBloke/Mistral-7B-Instruct-v0.2-GPTQ",
    device_map="auto",
    torch_dtype=torch.float16
)
tokenizer = AutoTokenizer.from_pretrained("TheBloke/Mistral-7B-Instruct-v0.2-GPTQ")

# ── Quantifier soi-même un modèle ─────────────────────────
from optimum.gptq import GPTQQuantizer
from datasets import load_dataset

# Charger le modèle original
model_original = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-Instruct-v0.3",
    torch_dtype=torch.float16,
    device_map="auto"
)

# Dataset de calibration (quelques centaines d'exemples suffisent)
dataset_calibration = load_dataset("wikitext", "wikitext-2-raw-v1", split="train[:512]")

quantizer = GPTQQuantizer(
    bits=4,                  # 4-bit (ou 8-bit)
    dataset=dataset_calibration,
    block_name_to_quantize="model.layers",
    model_seqlen=2048
)

model_gptq = quantizer.quantize_model(model_original, tokenizer)
model_gptq.save_pretrained("./mistral_gptq_4bit")
tokenizer.save_pretrained("./mistral_gptq_4bit")
```

## AWQ — Activation-Aware Weight Quantization

AWQ protège les poids les plus importants (activations élevées), donnant une meilleure qualité que GPTQ à parité de taille.

```python
# pip install autoawq

from awq import AutoAWQForCausalLM
from transformers import AutoTokenizer

# ── Utiliser un modèle AWQ pré-quantifié ──────────────────
model = AutoAWQForCausalLM.from_quantized(
    "TheBloke/Mistral-7B-Instruct-v0.2-AWQ",
    fuse_layers=True,      # fusion des couches → 30-50% plus rapide
    trust_remote_code=False,
    safetensors=True
)
tokenizer = AutoTokenizer.from_pretrained("TheBloke/Mistral-7B-Instruct-v0.2-AWQ")

# ── Quantifier soi-même ────────────────────────────────────
model = AutoAWQForCausalLM.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")

quant_config = {
    "zero_point": True,    # quantification symétrique ou asymétrique
    "q_group_size": 128,   # taille des groupes de quantification
    "w_bit": 4,            # bits de précision
    "version": "GEMM"      # algorithme de multiplication matricielle
}

model.quantize(tokenizer, quant_config=quant_config)
model.save_quantized("./mistral_awq_4bit")
tokenizer.save_pretrained("./mistral_awq_4bit")
```

## Comparaison des méthodes de quantification

| Méthode | VRAM 7B | Qualité | Vitesse inférence | Usage recommandé |
|---|---|---|---|---|
| **float16** | 14 GB | Référence | Référence | Quand tu as assez de VRAM |
| **BnB 8-bit** | 7 GB | Très proche | Légèrement plus lent | Fine-tuning (QLoRA) |
| **BnB 4-bit NF4** | 3.5 GB | Bonne | Similaire | Fine-tuning QLoRA |
| **GPTQ 4-bit** | 3.5 GB | Très bonne | Plus rapide | Inférence seule |
| **AWQ 4-bit** | 3.5 GB | Meilleure | La plus rapide | Inférence production |

## Choisir selon l'usage

```
Fine-tuning (QLoRA) :
  → BitsAndBytes 4-bit NF4 (compatible avec PEFT)

Inférence dev / prototypage :
  → BitsAndBytes (simple, pas de préparation)

Inférence production :
  → AWQ (meilleure qualité + vitesse)
  → GPTQ (bonne alternative)
  → vLLM charge les deux nativement

Trop peu de VRAM :
  → AWQ 4-bit + vLLM + fuse_layers=True
```

## Vérifier la VRAM utilisée

```python
import torch

if torch.cuda.is_available():
    print(f"VRAM allouée : {torch.cuda.memory_allocated() / 1e9:.2f} GB")
    print(f"VRAM réservée : {torch.cuda.memory_reserved() / 1e9:.2f} GB")
    print(f"VRAM totale : {torch.cuda.get_device_properties(0).total_memory / 1e9:.2f} GB")
```

> [!tip] TheBloke sur Hugging Face
> Le compte "TheBloke" sur Hugging Face propose des centaines de modèles pré-quantifiés en GPTQ et AWQ. Cherche "nom-du-modele GPTQ" ou "AWQ" pour trouver une version prête à l'emploi sans avoir à quantifier toi-même.

> [!warning] BitsAndBytes = Linux / GPU NVIDIA uniquement
> BitsAndBytes nécessite un GPU NVIDIA avec CUDA et tourne uniquement sur Linux. Pour macOS (Apple Silicon), utilise `llama.cpp` ou `mlx` à la place.
