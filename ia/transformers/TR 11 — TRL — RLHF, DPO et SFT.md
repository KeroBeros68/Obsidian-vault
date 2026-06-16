#ia #transformers #trl #rlhf #dpo #sft #alignement #avancé

## TRL — SFT, DPO et RLHF

TRL (Transformer Reinforcement Learning) est la librairie HuggingFace pour l'alignement des LLM : SFT (Supervised Fine-Tuning), DPO (Direct Preference Optimization) et RLHF (PPO).

## Installation

```bash
pip install trl peft bitsandbytes transformers accelerate datasets
```

## SFTTrainer — fine-tuning supervisé simplifié

TRL's `SFTTrainer` est plus simple que `Trainer` pour le fine-tuning de LLM — il gère automatiquement le formatage des conversations.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig
from trl import SFTTrainer, SFTConfig
from datasets import Dataset
import torch

# ── Dataset ───────────────────────────────────────────────
données = [
    {"messages": [
        {"role": "user",      "content": "Qu'est-ce que le RAG ?"},
        {"role": "assistant", "content": "Le RAG est une technique qui..."}
    ]},
    {"messages": [
        {"role": "system",    "content": "Tu es un expert en IA."},
        {"role": "user",      "content": "Explique LoRA."},
        {"role": "assistant", "content": "LoRA est une méthode de fine-tuning..."}
    ]},
    # ... beaucoup d'exemples
]
dataset = Dataset.from_list(données)

# ── Modèle + QLoRA ────────────────────────────────────────
modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True
)

tokenizer = AutoTokenizer.from_pretrained(modèle_id)
tokenizer.pad_token = tokenizer.eos_token
tokenizer.padding_side = "right"

model = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    quantization_config=bnb_config,
    device_map="auto"
)

lora_config = LoraConfig(
    r=16, lora_alpha=32, lora_dropout=0.05,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    bias="none", task_type="CAUSAL_LM"
)

# ── SFTConfig ─────────────────────────────────────────────
sft_config = SFTConfig(
    output_dir="./sft_output",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=2e-4,
    fp16=True,
    logging_steps=10,
    save_steps=100,
    optim="paged_adamw_8bit",
    gradient_checkpointing=True,
    max_seq_length=2048,
    packing=True,   # emballer plusieurs exemples courts dans une séquence → efficace
)

# ── SFTTrainer ────────────────────────────────────────────
trainer = SFTTrainer(
    model=model,
    args=sft_config,
    train_dataset=dataset,
    peft_config=lora_config,
    tokenizer=tokenizer,
    # SFTTrainer applique automatiquement le chat template !
)

trainer.train()
trainer.save_model("./sft_final")
```

## DPO — Direct Preference Optimization

DPO entraîne le modèle à préférer les "bonnes" réponses sur les "mauvaises" sans avoir besoin d'un Reward Model séparé.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model
from trl import DPOTrainer, DPOConfig
from datasets import Dataset
import torch

# ── Dataset de préférences ────────────────────────────────
# Chaque exemple : prompt + réponse choisie + réponse rejetée
données_préférences = [
    {
        "prompt": "Explique le RAG en 2 phrases.",
        "chosen": "Le RAG combine la recherche vectorielle et la génération. "
                  "Il permet aux LLM d'accéder à des connaissances externes.",
        "rejected": "Le RAG est une technique d'IA. C'est bien."
        # "rejected" est correct mais moins complet/qualitatif
    },
    {
        "prompt": "Qu'est-ce que LoRA ?",
        "chosen": "LoRA est une méthode de fine-tuning efficace qui n'entraîne "
                  "que 0.1 à 3% des paramètres via des matrices de rang bas.",
        "rejected": "LoRA ça veut dire Low-Rank Adaptation."
    },
    # ... 1000+ exemples
]
dataset_dpo = Dataset.from_list(données_préférences)

# ── Modèle SFT de base (point de départ du DPO) ───────────
modèle_sft = "mon-org/modele-sft"  # modèle déjà fine-tuné en SFT

tokenizer = AutoTokenizer.from_pretrained(modèle_sft)
tokenizer.pad_token = tokenizer.eos_token

model = AutoModelForCausalLM.from_pretrained(
    modèle_sft,
    torch_dtype=torch.float16,
    device_map="auto"
)

lora_config = LoraConfig(
    r=16, lora_alpha=32,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    bias="none", task_type="CAUSAL_LM"
)

model_dpo = get_peft_model(model, lora_config)

# ── DPOConfig ─────────────────────────────────────────────
dpo_config = DPOConfig(
    output_dir="./dpo_output",
    num_train_epochs=3,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=4,
    learning_rate=5e-5,            # plus bas que SFT
    beta=0.1,                      # régularisation KL — garde le modèle proche de la base
    max_length=1024,
    max_prompt_length=512,
    fp16=True,
    logging_steps=10,
    optim="paged_adamw_8bit"
)

# ── DPOTrainer ────────────────────────────────────────────
dpo_trainer = DPOTrainer(
    model=model_dpo,
    ref_model=None,   # None = utilise le modèle de base automatiquement
    args=dpo_config,
    train_dataset=dataset_dpo,
    tokenizer=tokenizer
)

dpo_trainer.train()
dpo_trainer.save_model("./dpo_final")
```

## RLAIF — générer le dataset de préférences avec un LLM

```python
from anthropic import Anthropic

client = Anthropic()

def comparer_réponses(prompt: str, réponse_a: str, réponse_b: str) -> str:
    """Utilise Claude pour choisir la meilleure réponse."""
    message = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=10,
        messages=[{
            "role": "user",
            "content": f"""Quelle réponse est meilleure ? Réponds uniquement "A" ou "B".

Prompt : {prompt}

Réponse A : {réponse_a}

Réponse B : {réponse_b}"""
        }]
    )
    return message.content[0].text.strip()

# Générer le dataset de préférences avec RLAIF
dataset_préférences = []
for exemple in mes_prompts:
    réponse_1 = modèle.générer(exemple)
    réponse_2 = modèle.générer(exemple)

    meilleure = comparer_réponses(exemple, réponse_1, réponse_2)

    dataset_préférences.append({
        "prompt":   exemple,
        "chosen":   réponse_1 if meilleure == "A" else réponse_2,
        "rejected": réponse_2 if meilleure == "A" else réponse_1
    })
```

## Pipeline complet SFT → DPO

```
Modèle de base (Mistral 7B)
        ↓ SFTTrainer (données instruction/réponse)
Modèle SFT spécialisé
        ↓ Générer 2 réponses par prompt
        ↓ RLAIF (Claude juge) → dataset de préférences
        ↓ DPOTrainer (beta=0.1)
Modèle aligné DPO
        ↓ merge_and_unload()
Modèle de production
```

> [!tip] DPO > RLHF pour la plupart des cas
> DPO donne des résultats proches de RLHF avec 3× moins de complexité — pas besoin de Reward Model séparé, pas de PPO instable. C'est l'approche recommandée en 2025.

> [!info] beta dans DPO
> `beta` contrôle la force de la régularisation KL — à quel point le modèle peut s'éloigner du modèle de référence. Valeur typique : 0.1 à 0.5. Trop bas = le modèle diverge. Trop haut = pas assez d'alignement.
