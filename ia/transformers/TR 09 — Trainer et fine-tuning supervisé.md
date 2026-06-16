#ia #transformers #trainer #fine-tuning #sft #pratique

## Trainer et fine-tuning supervisé

`Trainer` est l'API de haut niveau de Transformers pour l'entraînement. Elle gère automatiquement la boucle d'entraînement, l'évaluation, les checkpoints et les logs.

## Installation

```bash
pip install transformers datasets evaluate accelerate
```

## Pipeline complet de fine-tuning avec Trainer

```python
from transformers import (
    AutoTokenizer,
    AutoModelForSequenceClassification,
    TrainingArguments,
    Trainer,
    DataCollatorWithPadding
)
from datasets import load_dataset
import evaluate
import numpy as np

# ── 1. Charger le dataset ─────────────────────────────────
dataset = load_dataset("imdb")
# Structure : {"train": Dataset, "test": Dataset}
# Colonnes : {"text": "...", "label": 0 ou 1}

# ── 2. Charger le tokenizer ───────────────────────────────
modèle_id = "distilbert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)

def tokeniser(exemples):
    return tokenizer(exemples["text"], truncation=True, max_length=512)

dataset_tokenisé = dataset.map(tokeniser, batched=True)

# ── 3. Charger le modèle ──────────────────────────────────
model = AutoModelForSequenceClassification.from_pretrained(
    modèle_id,
    num_labels=2,                          # 2 classes : positif / négatif
    id2label={0: "NEGATIVE", 1: "POSITIVE"},
    label2id={"NEGATIVE": 0, "POSITIVE": 1}
)

# ── 4. Définir les métriques ──────────────────────────────
accuracy = evaluate.load("accuracy")
f1       = evaluate.load("f1")

def calculer_métriques(eval_pred):
    logits, labels = eval_pred
    prédictions    = np.argmax(logits, axis=-1)
    return {
        "accuracy": accuracy.compute(predictions=prédictions, references=labels)["accuracy"],
        "f1":       f1.compute(predictions=prédictions, references=labels, average="weighted")["f1"]
    }

# ── 5. Configurer l'entraînement ──────────────────────────
training_args = TrainingArguments(
    output_dir="./resultats",              # où sauvegarder les checkpoints
    num_train_epochs=3,                    # nombre d'epochs
    per_device_train_batch_size=16,        # batch size par GPU
    per_device_eval_batch_size=32,
    learning_rate=2e-5,                    # lr typique pour le fine-tuning BERT
    weight_decay=0.01,                     # régularisation L2
    evaluation_strategy="epoch",           # évaluer à chaque epoch
    save_strategy="epoch",                 # sauvegarder à chaque epoch
    load_best_model_at_end=True,           # charger le meilleur modèle
    metric_for_best_model="f1",            # critère de sélection du meilleur
    logging_steps=50,                      # logger toutes les 50 steps
    warmup_ratio=0.1,                      # 10% de warmup
    fp16=True,                             # mixed precision (GPU NVIDIA)
    report_to="none"                       # ou "wandb", "tensorboard"
)

# ── 6. Créer le Trainer ───────────────────────────────────
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset_tokenisé["train"],
    eval_dataset=dataset_tokenisé["test"],
    tokenizer=tokenizer,
    data_collator=DataCollatorWithPadding(tokenizer),
    compute_metrics=calculer_métriques
)

# ── 7. Entraîner ──────────────────────────────────────────
trainer.train()

# ── 8. Évaluer ────────────────────────────────────────────
résultats = trainer.evaluate()
print(résultats)
# → {"eval_loss": 0.23, "eval_accuracy": 0.934, "eval_f1": 0.933}

# ── 9. Sauvegarder le modèle final ───────────────────────
trainer.save_model("./modele_final")
tokenizer.save_pretrained("./modele_final")
```

## Fine-tuning pour la génération (SFT sur un LLM)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments, Trainer
from datasets import Dataset

# Préparer le dataset au format instruction/réponse
données = [
    {"texte": "<s>[INST] Explique le RAG. [/INST] RAG est une technique qui..."},
    {"texte": "<s>[INST] Qu'est-ce que LangChain ? [/INST] LangChain est un framework..."},
    # ... milliers d'exemples
]
dataset = Dataset.from_list(données)

modèle_id = "mistralai/Mistral-7B-Instruct-v0.3"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
tokenizer.pad_token = tokenizer.eos_token

def tokeniser_causal(exemples):
    encodings = tokenizer(
        exemples["texte"],
        truncation=True,
        max_length=2048,
        padding="max_length"
    )
    # Pour la génération, les labels = les input_ids (autoregressive)
    encodings["labels"] = encodings["input_ids"].copy()
    return encodings

dataset_tokenisé = dataset.map(tokeniser_causal, batched=True, remove_columns=["texte"])

import torch
model = AutoModelForCausalLM.from_pretrained(
    modèle_id,
    torch_dtype=torch.float16,
    device_map="auto"
)

training_args = TrainingArguments(
    output_dir="./mistral_ft",
    num_train_epochs=3,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,   # simule un batch de 16
    learning_rate=2e-5,
    fp16=True,
    save_steps=100,
    logging_steps=10,
    optim="adamw_torch"
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset_tokenisé,
    tokenizer=tokenizer
)

trainer.train()
```

## Callbacks — contrôler l'entraînement

```python
from transformers import TrainerCallback, EarlyStoppingCallback

# Early stopping — arrêter si pas d'amélioration
early_stopping = EarlyStoppingCallback(
    early_stopping_patience=3,    # arrêter après 3 epochs sans amélioration
    early_stopping_threshold=0.01 # amélioration minimale requise
)

# Callback custom
class LogCallback(TrainerCallback):
    def on_epoch_end(self, args, state, control, **kwargs):
        print(f"Epoch {state.epoch:.0f} terminée | loss: {state.log_history[-1].get('eval_loss', 'N/A'):.4f}")

trainer = Trainer(
    ...,
    callbacks=[early_stopping, LogCallback()]
)
```

## Paramètres TrainingArguments — guide

| Paramètre | Valeur recommandée | Notes |
|---|---|---|
| `learning_rate` | 2e-5 à 5e-5 | Pour BERT/RoBERTa |
| `learning_rate` | 1e-5 à 2e-4 | Pour LLM + LoRA |
| `num_train_epochs` | 3-5 | Pour classification |
| `per_device_train_batch_size` | 8-32 | Selon VRAM disponible |
| `gradient_accumulation_steps` | 4-16 | Compense un petit batch size |
| `warmup_ratio` | 0.06-0.1 | % des steps en warmup |
| `weight_decay` | 0.01 | Régularisation standard |
| `fp16` | True | Si GPU NVIDIA, économise VRAM |
| `bf16` | True | Si GPU Ampere+ (A100, H100), plus stable que fp16 |
| `gradient_checkpointing` | True | Économise VRAM au prix d'un recalcul |

> [!tip] gradient_accumulation_steps
> Si tu n'as pas assez de VRAM pour un batch de 32, utilise `per_device_train_batch_size=4` avec `gradient_accumulation_steps=8`. Le modèle verra effectivement des batches de 32 sans saturer la VRAM.

> [!warning] gradient_checkpointing + LoRA
> `gradient_checkpointing=True` économise la VRAM mais est incompatible avec certaines configurations LoRA. Si tu obtiens une erreur, essaie `model.enable_input_require_grads()` avant de créer le Trainer.
