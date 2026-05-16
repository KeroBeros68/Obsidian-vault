#ia #fine-tuning #openai #anthropic #pratique #api

## Fine-tuning en pratique (OpenAI et Anthropic)

Les APIs cloud permettent de fine-tuner sans gérer d'infrastructure GPU. Idéal pour commencer.

## Option 1 — OpenAI Fine-tuning API

### Modèles fine-tunables
```
gpt-4o-mini-2024-07-18    ← recommandé (qualité + coût)
gpt-4o-2024-08-06
gpt-3.5-turbo
```

### Étape 1 — Préparer le dataset JSONL

```python
import json

# Créer le fichier d'entraînement
exemples = [
    {
        "messages": [
            {"role": "system", "content": "Tu es un assistant support pour Acme Corp. Tu réponds en français, de manière concise et professionnelle."},
            {"role": "user", "content": "Comment retourner un produit ?"},
            {"role": "assistant", "content": "Pour retourner un produit, connectez-vous à votre espace client > Mes commandes > Retourner un article. Délai : 30 jours après réception. Remboursement sous 5-7 jours ouvrés."}
        ]
    },
    # ... 99 autres exemples
]

with open("training_data.jsonl", "w") as f:
    for exemple in exemples:
        f.write(json.dumps(exemple, ensure_ascii=False) + "\n")
```

### Étape 2 — Uploader le fichier

```python
from openai import OpenAI

client = OpenAI()

# Upload du fichier d'entraînement
with open("training_data.jsonl", "rb") as f:
    fichier = client.files.create(file=f, purpose="fine-tune")

print(f"Fichier uploadé : {fichier.id}")
# → file-abc123xyz

# Upload du fichier de validation (optionnel mais recommandé)
with open("validation_data.jsonl", "rb") as f:
    fichier_val = client.files.create(file=f, purpose="fine-tune")
```

### Étape 3 — Lancer l'entraînement

```python
# Créer le job de fine-tuning
job = client.fine_tuning.jobs.create(
    training_file=fichier.id,
    validation_file=fichier_val.id,
    model="gpt-4o-mini-2024-07-18",
    hyperparameters={
        "n_epochs": 3,           # nombre de passes sur le dataset
        "batch_size": "auto",    # ou un entier
        "learning_rate_multiplier": "auto"
    },
    suffix="support-acme"  # nom personnalisé du modèle
)

print(f"Job créé : {job.id}")
# → ftjob-abc123

# Surveiller la progression
import time
while True:
    job = client.fine_tuning.jobs.retrieve(job.id)
    print(f"Statut : {job.status}")
    if job.status in ["succeeded", "failed"]:
        break
    time.sleep(60)

print(f"Modèle fine-tuné : {job.fine_tuned_model}")
# → ft:gpt-4o-mini-2024-07-18:acme:support-acme:abc123
```

### Étape 4 — Utiliser le modèle fine-tuné

```python
réponse = client.chat.completions.create(
    model="ft:gpt-4o-mini-2024-07-18:acme:support-acme:abc123",
    messages=[
        {"role": "system", "content": "Tu es un assistant support pour Acme Corp."},
        {"role": "user", "content": "Comment annuler ma commande ?"}
    ]
)
print(réponse.choices[0].message.content)
```

### Coûts OpenAI Fine-tuning (ordre de grandeur)

```
Entraînement gpt-4o-mini : ~$0.003 / 1000 tokens d'entraînement
1000 exemples × 500 tokens = 500 000 tokens → ~$1.50 par epoch
3 epochs → ~$4.50 pour l'entraînement complet

Inférence modèle fine-tuné :
  Input  : ~$0.30 / 1M tokens (vs $0.15 pour le modèle de base)
  Output : ~$1.20 / 1M tokens (vs $0.60 pour le modèle de base)
```

---

## Option 2 — Anthropic Fine-tuning (Claude)

### Disponibilité

Le fine-tuning de Claude est disponible pour les clients Enterprise via l'API Anthropic. Contacter Anthropic pour l'accès.

```
Modèles fine-tunables (selon disponibilité) :
  claude-haiku-4-5  ← le plus accessible pour le fine-tuning
  claude-sonnet     ← selon accord Enterprise
```

### Format du dataset (identique à OpenAI)

```jsonl
{"messages": [{"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}
```

> [!info] Fine-tuning Claude en 2025
> Le fine-tuning Claude est en accès limité. Consulter la documentation officielle sur docs.anthropic.com/fine-tuning pour les dernières informations sur la disponibilité et le processus.

---

## Option 3 — Fine-tuning open-source avec Unsloth

Pour fine-tuner des modèles open-source (Mistral, LLaMA, Qwen...) à moindre coût.

```python
from unsloth import FastLanguageModel
from trl import SFTTrainer
from transformers import TrainingArguments
from datasets import load_dataset

# Charger le modèle de base avec LoRA optimisé
modèle, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/mistral-7b-instruct-v0.3",
    max_seq_length=2048,
    load_in_4bit=True  # QLoRA
)

# Ajouter les adaptateurs LoRA
modèle = FastLanguageModel.get_peft_model(
    modèle,
    r=16,
    lora_alpha=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    lora_dropout=0,
    bias="none"
)

# Charger ton dataset
dataset = load_dataset("json", data_files="training_data.jsonl")

# Entraîner
trainer = SFTTrainer(
    model=modèle,
    tokenizer=tokenizer,
    train_dataset=dataset["train"],
    dataset_text_field="text",
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        num_train_epochs=3,
        learning_rate=2e-4,
        output_dir="./fine_tuned_model",
        logging_steps=10
    )
)

trainer.train()
modèle.save_pretrained("./fine_tuned_model")
```

### Plateformes cloud pour fine-tuning open-source

| Plateforme | Points forts | Coût indicatif |
|---|---|---|
| **Google Colab Pro+** | Accessible, GPU A100 | ~$50/mois |
| **Modal** | Serverless Python, facile | ~$1-5/heure GPU |
| **RunPod** | GPU à la demande, peu cher | ~$0.5-2/heure GPU |
| **Lambda Labs** | GPU dédié, stable | ~$1-3/heure GPU |
| **Hugging Face AutoTrain** | No-code, simple | Variable |

> [!tip] Commencer avec Google Colab + Unsloth
> Google Colab Pro+ + Unsloth est la façon la plus accessible de faire du fine-tuning. Unsloth optimise l'utilisation mémoire pour permettre de fine-tuner des modèles 7B sur un seul GPU T4/A100.
