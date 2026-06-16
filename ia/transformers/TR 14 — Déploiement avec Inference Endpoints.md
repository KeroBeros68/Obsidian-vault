#ia #transformers #déploiement #inference-endpoints #huggingface #production

## Déploiement avec Hugging Face Inference Endpoints

Hugging Face Inference Endpoints permet de déployer n'importe quel modèle du Hub en API REST managée, sans gérer l'infrastructure GPU.

## Les options de déploiement

```
Option A — Inference API (public, gratuit)
  → API publique HuggingFace, rate limitée
  → Pas pour la production
  → Test rapide uniquement

Option B — Inference Endpoints (managé, payant)
  → GPU dédié (A10, A100, H100...)
  → API privée avec authentification
  → Auto-scaling optionnel
  → Idéal production sans gérer l'infra

Option C — Spaces (Gradio/Streamlit)
  → UI web hébergée
  → GPU partagé ou dédié
  → Idéal pour les démos

Option D — Self-hosted (vLLM, TGI)
  → Contrôle total
  → Tes propres GPU (cloud ou on-premise)
  → Voir fiche vLLM
```

## Text Generation Inference (TGI) — le serving HuggingFace

TGI est le serveur d'inférence optimisé de Hugging Face. Il propulse les Inference Endpoints.

### Lancer TGI avec Docker

```bash
# Mistral 7B en fp16 sur GPU
docker run --gpus all \
    --shm-size 1g \
    -p 8080:80 \
    -v $(pwd)/models:/data \
    ghcr.io/huggingface/text-generation-inference:latest \
    --model-id mistralai/Mistral-7B-Instruct-v0.3 \
    --dtype float16 \
    --max-input-length 4096 \
    --max-total-tokens 8192

# Avec quantification (GPU moins puissant)
docker run --gpus all -p 8080:80 \
    ghcr.io/huggingface/text-generation-inference:latest \
    --model-id mistralai/Mistral-7B-Instruct-v0.3 \
    --quantize bitsandbytes-nf4
```

### Appeler TGI depuis Python

```python
import requests

# API compatible OpenAI (depuis TGI 2.0)
import openai

client = openai.OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="no-key"
)

# Appel standard OpenAI
réponse = client.chat.completions.create(
    model="tgi",
    messages=[
        {"role": "system", "content": "Tu es un assistant expert."},
        {"role": "user",   "content": "Explique le RAG."}
    ],
    max_tokens=500,
    stream=False
)
print(réponse.choices[0].message.content)

# Avec streaming
for chunk in client.chat.completions.create(
    model="tgi",
    messages=[{"role": "user", "content": "Explique LoRA."}],
    max_tokens=500,
    stream=True
):
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

## Inference Endpoints HuggingFace — déploiement cloud

```python
from huggingface_hub import InferenceClient

# Utiliser un endpoint privé HuggingFace
client = InferenceClient(
    model="https://ton-endpoint.aws.endpoints.huggingface.cloud",
    token="hf_xxxxxxxxxxxxx"
)

# Génération de texte
réponse = client.text_generation(
    "Explique le RAG en 2 phrases.",
    max_new_tokens=200,
    temperature=0.7
)
print(réponse)

# Chat completion
messages = [
    {"role": "system", "content": "Tu es un assistant expert."},
    {"role": "user",   "content": "Qu'est-ce que LoRA ?"}
]

réponse = client.chat_completion(messages, max_tokens=300)
print(réponse.choices[0].message.content)
```

## Intégration LangChain avec TGI / Endpoints

```python
from langchain_community.llms import HuggingFaceEndpoint
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Endpoint TGI local ou HuggingFace Cloud
llm = HuggingFaceEndpoint(
    endpoint_url="http://localhost:8080",   # ou URL HF Endpoints
    huggingfacehub_api_token="hf_xxx",
    max_new_tokens=512,
    temperature=0.7,
    repetition_penalty=1.1
)

# Utiliser normalement dans une chain LangChain
chain = (
    ChatPromptTemplate.from_messages([("human", "{question}")])
    | llm
    | StrOutputParser()
)

réponse = chain.invoke({"question": "Qu'est-ce que le RAG ?"})
```

## Uploader un modèle fine-tuné sur le Hub

```python
from huggingface_hub import HfApi, login

login(token="hf_xxxxxxxxxxxxx")
api = HfApi()

# Créer un repo
api.create_repo(
    repo_id="mon-org/mistral-7b-rag-expert",
    private=True,        # repo privé
    repo_type="model"
)

# Uploader depuis un dossier local
api.upload_folder(
    folder_path="./modele_final",
    repo_id="mon-org/mistral-7b-rag-expert",
    repo_type="model"
)

# Ou uploader directement depuis le Trainer
from transformers import TrainingArguments

training_args = TrainingArguments(
    ...,
    push_to_hub=True,
    hub_model_id="mon-org/mistral-7b-rag-expert",
    hub_private_repo=True
)
# trainer.push_to_hub() après l'entraînement
```

## Gradio — démo rapide

```python
import gradio as gr
from transformers import pipeline

# Créer un pipeline
pipe = pipeline("text-generation", model="mistralai/Mistral-7B-Instruct-v0.3",
                device_map="auto")

def générer(message, historique):
    messages = [{"role": "system", "content": "Tu es un assistant utile."}]
    for h in historique:
        messages.append({"role": "user",      "content": h[0]})
        messages.append({"role": "assistant", "content": h[1]})
    messages.append({"role": "user", "content": message})

    # Appliquer le chat template
    from transformers import AutoTokenizer
    tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")
    prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)

    résultat = pipe(prompt, max_new_tokens=300, return_full_text=False)
    return résultat[0]["generated_text"]

# Interface Gradio
demo = gr.ChatInterface(générer, title="Mon assistant LLM")
demo.launch(server_port=7860, share=False)
```

> [!tip] TGI pour la prod, pipeline() pour le dev
> En développement, `pipeline()` est plus simple. En production, TGI offre des optimisations importantes : continuous batching, paged attention, tensor parallelism. Le passage de l'un à l'autre est transparent via l'API OpenAI-compatible.

> [!info] Continuous batching
> TGI implémente le continuous batching : au lieu de traiter les requêtes une par une, il les groupe dynamiquement pour maximiser l'utilisation GPU. Sur un GPU A100, le throughput peut être 3-10× supérieur à un simple serving PyTorch.
