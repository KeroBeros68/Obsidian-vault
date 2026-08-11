#ia #transformers #huggingface #bases #définition

## Qu'est-ce que Hugging Face Transformers ?

`transformers` est la librairie Python open-source de Hugging Face qui donne accès à des milliers de modèles pré-entraînés — LLM, embeddings, classification, NER, résumé, traduction, vision...

## La place dans l'écosystème

```
HUGGING FACE HUB
  → dépôt de 500k+ modèles open-source

TRANSFORMERS (librairie)
  → charge, utilise et fine-tune ces modèles

TOKENIZERS
  → tokenisation ultra-rapide (Rust)

DATASETS
  → charge et prépare les datasets

PEFT
  → LoRA, QLoRA, adaptateurs

TRL
  → SFT, RLHF, DPO, PPO

ACCELERATE
  → entraînement multi-GPU / distribué

BITSANDBYTES
  → quantification 4-bit / 8-bit

SENTENCE-TRANSFORMERS
  → embeddings sémantiques optimisés
```

## Ce que Transformers fait concrètement

```python
# Sans Transformers : API cloud (boîte noire, coût par token)
réponse = anthropic.messages.create(model="claude-sonnet-4-20250514", ...)

# Avec Transformers : modèle local (contrôle total, gratuit)
from transformers import AutoModelForCausalLM, AutoTokenizer
model     = AutoModelForCausalLM.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")
# → Même modèle, zéro API, zéro coût par token
```

## L'écosystème Hugging Face en résumé

```
Hub                   : héberge les modèles et datasets
transformers          : charge et utilise les modèles
tokenizers            : tokenisation rapide (Rust)
datasets              : datasets prêts à l'emploi
peft                  : fine-tuning efficace (LoRA, QLoRA)
trl                   : alignement (SFT, DPO, RLHF)
accelerate            : multi-GPU, mixed precision
bitsandbytes          : quantification 4-bit / 8-bit
sentence-transformers : embeddings sémantiques
evaluate              : métriques d'évaluation
```

## Installation

```bash
# Core
pip install transformers

# Avec PyTorch (recommandé)
pip install transformers torch

# Avec tous les extras utiles
pip install transformers[torch] datasets peft trl accelerate bitsandbytes

# Sentence Transformers (embeddings)
pip install sentence-transformers

# Pour GPU NVIDIA (CUDA)
pip install torch --index-url https://download.pytorch.org/whl/cu121
```

## Hugging Face Hub — où vivent les modèles

```python
from huggingface_hub import login, snapshot_download

# Se connecter (token sur huggingface.co/settings/tokens)
login(token="hf_xxxxxxxxxxxx")

# Télécharger un modèle entier localement
snapshot_download("mistralai/Mistral-7B-Instruct-v0.3", local_dir="./mistral-7b")

# Les modèles sont mis en cache par défaut dans :
# ~/.cache/huggingface/hub/
```

> [!warning] "You are trying to access a gated model" : deux étapes, pas une seule
> Certains modèles (Llama, Mistral...) exigent d'accepter leur licence sur la page du modèle sur huggingface.co **en plus** de la connexion (`login()` ou `huggingface-cli login`). Se connecter sans avoir accepté la licence sur la page du modèle produit la même erreur — les deux étapes sont indépendantes et toutes deux nécessaires.

## Les grandes familles de modèles sur le Hub

```
LLM (génération de texte) :
  Mistral, LLaMA, Qwen, Gemma, Phi, Falcon, MPT...

Embeddings :
  sentence-transformers/*, BAAI/bge-*, intfloat/e5-*...

Classification :
  distilbert-base-*, roberta-*, camembert-*...

NER (entités nommées) :
  Jean-Baptiste/*, dslim/bert-base-NER...

Résumé :
  facebook/bart-large-cnn, google/pegasus-*...

Traduction :
  Helsinki-NLP/opus-mt-*, facebook/nllb-*...

Vision + Texte (multimodal) :
  llava-hf/*, Qwen/Qwen2-VL-*...

Audio :
  openai/whisper-*, facebook/wav2vec2-*...
```

## Compatibilité avec LangChain

```python
# HuggingFaceEmbeddings utilise sentence-transformers sous le capot
from langchain_community.embeddings import HuggingFaceEmbeddings
embeddings = HuggingFaceEmbeddings(model_name="...")
# → Appelle sentence_transformers.SentenceTransformer en interne

# HuggingFacePipeline charge un modèle transformers dans LangChain
from langchain_community.llms import HuggingFacePipeline
from transformers import pipeline

pipe = pipeline("text-generation", model="mistralai/Mistral-7B-Instruct-v0.3")
llm  = HuggingFacePipeline(pipeline=pipe)
# → Utilisable dans n'importe quelle chain LangChain
```

> [!tip] Transformers = le moteur sous le capot
> Quand tu utilises `HuggingFaceEmbeddings` dans LangChain, tu utilises déjà `sentence-transformers` qui utilise `transformers`. Comprendre la librairie te donne le contrôle sur ce qui se passe réellement.

> [!info] PyTorch vs TensorFlow
> Hugging Face supporte PyTorch et TensorFlow. Dans la pratique, **95% de l'écosystème MLOps utilise PyTorch**. Toutes les fiches de ce module supposent PyTorch.

> [!tip] Comprendre ce que la librairie charge
> `AutoModelForCausalLM.from_pretrained(...)` instancie une architecture Transformer complète (embeddings, blocs d'attention empilés, couche de sortie). Voir [[TR 01 — L'architecture Transformer & le mécanisme d'attention]] pour ce que ces quelques lignes construisent réellement.
