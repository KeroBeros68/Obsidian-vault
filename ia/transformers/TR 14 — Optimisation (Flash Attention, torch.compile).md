#ia #transformers #flash-attention #optimisation #performance #avancé

## Optimisation — Flash Attention et torch.compile

Deux techniques complémentaires pour accélérer l'inférence et l'entraînement sans dégrader la qualité.

## Flash Attention 2 — attention efficace en mémoire

Flash Attention réimplemente le mécanisme d'attention en CUDA pour éviter de matérialiser la matrice d'attention complète en VRAM.

```
Attention standard :
  Matrice d'attention (seq × seq) = O(n²) en VRAM
  Pour seq=4096 : 4096² = 16M éléments → ~64MB par couche

Flash Attention 2 :
  Calcul par blocs (tiling) → O(n) en VRAM
  Même résultat exact, mais beaucoup moins de VRAM et plus rapide
  Gain typique : 2-4× plus rapide, 5-20× moins de VRAM pour l'attention
```

### Installation

```bash
# Nécessite GPU Ampere+ (A100, A10, RTX 3090, 4090...)
pip install flash-attn --no-build-isolation
```

### Activer Flash Attention 2

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)

# Activer Flash Attention 2 avec attn_implementation
model = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    torch_dtype=torch.bfloat16,      # bfloat16 recommandé avec FA2
    device_map="auto",
    attn_implementation="flash_attention_2"   # ← une seule ligne !
)

# Vérifier
print(model.config._attn_implementation)   # → "flash_attention_2"
```

### Benchmark Flash Attention 2

```python
import torch
import time
from transformers import AutoModelForCausalLM, AutoTokenizer

def benchmark_génération(model, tokenizer, prompt: str, n: int = 5) -> float:
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    latences = []
    for _ in range(n):
        start = time.time()
        with torch.no_grad():
            model.generate(**inputs, max_new_tokens=200, do_sample=False)
        latences.append(time.time() - start)
    return sum(latences) / len(latences)

# Comparer
model_standard = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-Instruct-v0.3", torch_dtype=torch.bfloat16, device_map="auto"
)
model_fa2 = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-Instruct-v0.3", torch_dtype=torch.bfloat16,
    device_map="auto", attn_implementation="flash_attention_2"
)

prompt = "Explique le RAG en détail avec des exemples concrets."
t_standard = benchmark_génération(model_standard, tokenizer, prompt)
t_fa2      = benchmark_génération(model_fa2,      tokenizer, prompt)

print(f"Standard  : {t_standard:.2f}s")
print(f"FA2       : {t_fa2:.2f}s")
print(f"Speedup   : {t_standard/t_fa2:.1f}×")
```

## torch.compile — compilation du graphe de calcul

`torch.compile` (PyTorch 2.0+) compile le modèle pour l'optimiser spécifiquement pour le GPU cible.

```python
import torch
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-Instruct-v0.3",
    torch_dtype=torch.bfloat16,
    device_map="auto",
    attn_implementation="flash_attention_2"
)

# Compiler le modèle
model = torch.compile(
    model,
    mode="reduce-overhead",   # modes : "default", "reduce-overhead", "max-autotune"
    fullgraph=False           # True = plus d'optimisations mais moins compatible
)
# ⚠️ Premier appel = compilation (30s à 5min selon le modèle)
# Les appels suivants sont beaucoup plus rapides

# Mode max-autotune — benchmarke plusieurs optimisations et choisit le meilleur
# Plus long à compiler, meilleure performance en production
model_ultra_rapide = torch.compile(model, mode="max-autotune")
```

### Modes torch.compile

| Mode | Temps compilation | Speedup | Compatibilité |
|---|---|---|---|
| `"default"` | Rapide | +10-20% | Meilleure |
| `"reduce-overhead"` | Moyen | +20-40% | Bonne |
| `"max-autotune"` | Lent (5-10min) | +30-50% | Moins bonne |

## Optimisations combinées

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)

# Stack d'optimisations complet
model = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    torch_dtype=torch.bfloat16,                  # 1. precision optimale
    device_map="auto",                            # 2. distribution GPU auto
    attn_implementation="flash_attention_2"       # 3. Flash Attention 2
)
model = torch.compile(model, mode="reduce-overhead")  # 4. compilation JIT

# Inférence optimisée
model.eval()

# Désactiver les calculs de gradient en inférence
with torch.no_grad():
    # KV cache activé par défaut → réutilise les clés/valeurs précédentes
    inputs = tokenizer("Bonjour !", return_tensors="pt").to(model.device)
    outputs = model.generate(
        **inputs,
        max_new_tokens=200,
        use_cache=True      # KV cache → essentiel pour la vitesse de génération
    )
```

## Accelerate — multi-GPU et mixed precision

```python
from accelerate import Accelerator
from transformers import AutoModelForCausalLM, TrainingArguments

# Initialiser Accelerate
accelerator = Accelerator(
    mixed_precision="bf16",      # ou "fp16"
    gradient_accumulation_steps=4
)

# Le modèle et les données sont automatiquement placés sur les bons devices
model, optimizer, dataloader = accelerator.prepare(model, optimizer, dataloader)

# Boucle d'entraînement
for batch in dataloader:
    with accelerator.accumulate(model):
        outputs = model(**batch)
        loss = outputs.loss
        accelerator.backward(loss)
        optimizer.step()
        optimizer.zero_grad()
```

## Better Transformers — optimisation CPU/GPU

```python
from optimum.bettertransformer import BetterTransformer

# Convertir le modèle pour utiliser les kernels optimisés PyTorch
model_bt = BetterTransformer.transform(model)

# Utilisation identique
with torch.no_grad():
    outputs = model_bt.generate(**inputs, max_new_tokens=200)

# Revenir au modèle original si nécessaire
model_original = BetterTransformer.reverse(model_bt)
```

## Récapitulatif — gains attendus

```
Optimisation           Gain vitesse   Impact VRAM   Compatibilité
──────────────────────────────────────────────────────────────────
bfloat16               2×             -50%          GPU Ampere+
Flash Attention 2      2-4×           -80% attention GPU Ampere+
torch.compile          1.2-1.5×       Neutre        PyTorch 2.0+
use_cache=True         3-10× (génération) +         Toujours actif
QLoRA (fine-tuning)    -              -75%          GPU NVIDIA

Combiné (FA2 + compile + bf16) : 3-6× vs baseline fp32
```

> [!tip] Ordre d'optimisation recommandé
> 1. `torch_dtype=torch.bfloat16` (gratuit, toujours)
> 2. `attn_implementation="flash_attention_2"` (si GPU Ampere+)
> 3. `torch.compile(mode="reduce-overhead")` (si déploiement long terme)
> 4. Quantification AWQ/GPTQ (si VRAM insuffisante)

> [!warning] torch.compile n'est pas magique
> Sur des petits modèles ou des batches de 1, le gain est faible voire nul. Le gain est significatif sur des batches de taille 8+ et des modèles > 1B paramètres.
