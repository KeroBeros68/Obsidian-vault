#ia #transformers #peft #lora #qlora #fine-tuning #avancé

## PEFT et LoRA

PEFT (Parameter-Efficient Fine-Tuning) est la librairie HuggingFace qui implémente LoRA, QLoRA et d'autres méthodes de fine-tuning efficace. Elle permet de fine-tuner de grands modèles avec une fraction des ressources.

## Installation

```bash
pip install peft bitsandbytes accelerate transformers
```

## LoRA — Low-Rank Adaptation

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model, TaskType
import torch

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"

# Charger le modèle de base
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
tokenizer.pad_token = tokenizer.eos_token

model = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    torch_dtype=torch.float16,
    device_map="auto"
)

# ── Configurer LoRA ───────────────────────────────────────
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,    # type de tâche
    r=16,                            # rang des matrices d'adaptation
    lora_alpha=32,                   # facteur de scaling (souvent = 2×r)
    lora_dropout=0.05,               # dropout pour régularisation
    bias="none",                     # ne pas adapter les biais
    target_modules=[                 # quelles couches adapter
        "q_proj", "k_proj", "v_proj", "o_proj",   # attention
        "gate_proj", "up_proj", "down_proj"        # MLP (optionnel)
    ]
)

# ── Appliquer LoRA au modèle ──────────────────────────────
model_lora = get_peft_model(model, lora_config)
model_lora.print_trainable_parameters()
# → trainable params: 41,943,040 || all params: 3.8B || trainable%: 1.11%
# Seulement 1.1% des paramètres sont entraînés !
```

## QLoRA — LoRA + quantification 4-bit

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training
import torch

# ── Configuration 4-bit ───────────────────────────────────
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,                     # charger en 4-bit
    bnb_4bit_quant_type="nf4",             # NormalFloat4 (recommandé)
    bnb_4bit_compute_dtype=torch.bfloat16, # calcul en bfloat16
    bnb_4bit_use_double_quant=True         # double quantification (économise +15% VRAM)
)

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"

# ── Charger le modèle en 4-bit ───────────────────────────
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
tokenizer.pad_token = tokenizer.eos_token

model = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    quantization_config=bnb_config,
    device_map="auto"
)

# ── Préparer pour le k-bit training (obligatoire) ─────────
model = prepare_model_for_kbit_training(model)

# ── Configurer LoRA ───────────────────────────────────────
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=16,
    lora_alpha=32,
    lora_dropout=0.05,
    bias="none",
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"]
)

model_qlora = get_peft_model(model, lora_config)
model_qlora.print_trainable_parameters()
# → Mistral 7B → ~14GB en fp16 → ~4.5GB en QLoRA !
```

## Entraîner avec Trainer + LoRA

```python
from transformers import TrainingArguments, Trainer, DataCollatorForLanguageModeling
from datasets import Dataset

# Dataset de fine-tuning
données = [
    {"text": "<s>[INST] Qu'est-ce que le RAG ? [/INST] Le RAG est..."},
    # ...
]
dataset = Dataset.from_list(données)

def tokeniser(exemples):
    result = tokenizer(exemples["text"], truncation=True, max_length=2048)
    result["labels"] = result["input_ids"].copy()
    return result

dataset_ft = dataset.map(tokeniser, batched=True, remove_columns=["text"])

training_args = TrainingArguments(
    output_dir="./mistral_qlora",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,   # batch effectif = 16
    learning_rate=2e-4,              # lr plus élevé avec LoRA
    fp16=True,
    logging_steps=10,
    save_steps=100,
    optim="paged_adamw_8bit",        # optimiseur adapté au QLoRA
    gradient_checkpointing=True,     # économiser la VRAM
    warmup_ratio=0.03
)

trainer = Trainer(
    model=model_qlora,
    args=training_args,
    train_dataset=dataset_ft,
    tokenizer=tokenizer,
    data_collator=DataCollatorForLanguageModeling(tokenizer, mlm=False)
)

trainer.train()
```

## Sauvegarder et charger les adaptateurs LoRA

```python
# Sauvegarder uniquement les adaptateurs (quelques MB !)
model_qlora.save_pretrained("./adaptateurs_lora")
tokenizer.save_pretrained("./adaptateurs_lora")

# ── Charger pour l'inférence ──────────────────────────────
from peft import PeftModel

# 1. Charger le modèle de base
model_base = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    quantization_config=bnb_config,  # ou sans quantification
    device_map="auto"
)

# 2. Appliquer les adaptateurs LoRA
model_ft = PeftModel.from_pretrained(model_base, "./adaptateurs_lora")
model_ft = model_ft.merge_and_unload()   # fusionner LoRA dans le modèle → plus rapide en inférence

# 3. Inférence normale
inputs = tokenizer("Qu'est-ce que le RAG ?", return_tensors="pt").to(model_ft.device)
with torch.no_grad():
    outputs = model_ft.generate(**inputs, max_new_tokens=200)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

## Choisir les target_modules

```python
# Voir les modules disponibles du modèle
for name, module in model.named_modules():
    print(name)

# Modules typiques selon l'architecture :

# Mistral / LLaMA :
target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                 "gate_proj", "up_proj", "down_proj"]

# Falcon :
target_modules=["query_key_value", "dense", "dense_h_to_4h", "dense_4h_to_h"]

# GPT-2 / GPT-Neo :
target_modules=["c_attn", "c_proj"]

# BERT / RoBERTa :
target_modules=["query", "key", "value", "dense"]

# Phi-2 :
target_modules=["q_proj", "k_proj", "v_proj", "dense",
                 "fc1", "fc2"]
```

## Comparer les méthodes PEFT

| Méthode | Params entraînés | VRAM | Qualité | Usage |
|---|---|---|---|---|
| **Full fine-tuning** | 100% | Très élevée | Maximale | Lab IA |
| **LoRA** | 0.1-3% | Réduite | Très proche full | Standard |
| **QLoRA** | 0.1-3% | Minimale | Légèrement sous LoRA | GPU limité |
| **Prefix Tuning** | < 0.1% | Très faible | Moins bon | Cas spéciaux |
| **Prompt Tuning** | < 0.01% | Minimal | Limité | Prototypage |

> [!tip] r=16, lora_alpha=32 — la config de départ
> Ces valeurs sont un bon point de départ pour la plupart des modèles. Si les résultats sont insuffisants, essaie `r=32, lora_alpha=64`. Si tu manques de VRAM, descends à `r=8, lora_alpha=16`.

> [!warning] merge_and_unload() avant de déployer
> En production, toujours fusionner les adaptateurs dans le modèle avec `merge_and_unload()`. Le modèle fusionné est légèrement plus lent à charger mais beaucoup plus rapide en inférence car il n'y a plus de calcul LoRA séparé.
