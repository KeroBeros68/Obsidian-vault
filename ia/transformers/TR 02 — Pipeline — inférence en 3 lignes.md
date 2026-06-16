#ia #transformers #pipeline #inférence #bases

## Pipeline — inférence en 3 lignes

`pipeline()` est l'API de haut niveau de Transformers. Elle abstrait toute la complexité : chargement du modèle, tokenisation, inférence, décodage.

## Syntaxe de base

```python
from transformers import pipeline

# 3 lignes pour de l'inférence
pipe = pipeline("task", model="nom-du-modele")
résultat = pipe("entrée")
print(résultat)
```

## Les tâches disponibles

```python
from transformers import pipeline

# ── Génération de texte ───────────────────────────────────
pipe = pipeline("text-generation", model="mistralai/Mistral-7B-Instruct-v0.3")
résultat = pipe("Explique le RAG en 3 phrases :", max_new_tokens=200)
print(résultat[0]["generated_text"])

# ── Résumé ────────────────────────────────────────────────
pipe = pipeline("summarization", model="facebook/bart-large-cnn")
résultat = pipe("Long texte à résumer...", max_length=100, min_length=30)
print(résultat[0]["summary_text"])

# ── Classification de texte ───────────────────────────────
pipe = pipeline("text-classification", model="distilbert-base-uncased-finetuned-sst-2-english")
résultat = pipe("This product is amazing!")
print(résultat)   # → [{"label": "POSITIVE", "score": 0.9998}]

# ── Classification zero-shot (sans fine-tuning) ───────────
pipe = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
résultat = pipe(
    "Je veux retourner mon article défectueux.",
    candidate_labels=["retour", "livraison", "garantie", "paiement"]
)
print(résultat["labels"][0])   # → "retour"
print(résultat["scores"][0])   # → 0.95

# ── NER (entités nommées) ─────────────────────────────────
pipe = pipeline("ner", model="dslim/bert-base-NER", aggregation_strategy="simple")
résultat = pipe("Apple est basée à Cupertino en Californie.")
for entité in résultat:
    print(f"{entité['word']} → {entité['entity_group']} ({entité['score']:.2f})")
# → Apple → ORG (0.99)
# → Cupertino → LOC (0.98)
# → Californie → LOC (0.97)

# ── Question-Answering ────────────────────────────────────
pipe = pipeline("question-answering", model="deepset/roberta-base-squad2")
résultat = pipe(
    question="Quelle est la durée de la garantie ?",
    context="Tous nos produits sont garantis 2 ans pièces et main d'œuvre."
)
print(résultat["answer"])   # → "2 ans"
print(résultat["score"])    # → 0.97

# ── Traduction ────────────────────────────────────────────
pipe = pipeline("translation_fr_to_en", model="Helsinki-NLP/opus-mt-fr-en")
résultat = pipe("Bonjour, comment puis-je vous aider ?")
print(résultat[0]["translation_text"])   # → "Hello, how can I help you?"

# ── Génération d'embeddings ───────────────────────────────
pipe = pipeline("feature-extraction", model="sentence-transformers/all-MiniLM-L6-v2")
embeddings = pipe("Hello world", return_tensors=False)
# → vecteur de 384 dimensions

# ── Transcription audio ───────────────────────────────────
pipe = pipeline("automatic-speech-recognition", model="openai/whisper-base")
résultat = pipe("audio.mp3")
print(résultat["text"])

# ── Remplissage de masque ─────────────────────────────────
pipe = pipeline("fill-mask", model="camembert-base")
résultats = pipe("Le chat <mask> sur le tapis.")
for r in résultats[:3]:
    print(f"{r['token_str']:15} → {r['score']:.3f}")
```

## Paramètres GPU et performance

```python
# Utiliser le GPU si disponible
import torch

device = 0 if torch.cuda.is_available() else -1   # 0=GPU, -1=CPU

pipe = pipeline(
    "text-generation",
    model="mistralai/Mistral-7B-Instruct-v0.3",
    device=device,
    torch_dtype=torch.float16   # float16 = moitié moins de VRAM
)

# Avec device_map="auto" — distribue sur tous les GPU disponibles
pipe = pipeline(
    "text-generation",
    model="mistralai/Mistral-7B-Instruct-v0.3",
    device_map="auto",
    torch_dtype=torch.bfloat16
)
```

## Paramètres de génération

```python
pipe = pipeline("text-generation", model="mistralai/Mistral-7B-Instruct-v0.3")

résultat = pipe(
    "Explique le machine learning :",
    max_new_tokens=300,     # max de tokens à générer
    min_new_tokens=50,      # min de tokens à générer
    temperature=0.7,        # créativité (0=déterministe, 1=créatif)
    top_p=0.9,              # nucleus sampling
    top_k=50,               # top-k sampling
    repetition_penalty=1.1, # pénalise les répétitions
    do_sample=True,         # activer le sampling (sinon greedy)
    num_return_sequences=1, # nombre de réponses générées
    return_full_text=False  # False = retourne seulement la génération, pas le prompt
)
```

## Pipeline dans LangChain

```python
from transformers import pipeline
from langchain_community.llms import HuggingFacePipeline

# Créer le pipeline Transformers
pipe = pipeline(
    "text-generation",
    model="mistralai/Mistral-7B-Instruct-v0.3",
    device_map="auto",
    torch_dtype=torch.float16,
    max_new_tokens=512
)

# Envelopper dans LangChain
llm = HuggingFacePipeline(pipeline=pipe)

# Utiliser dans une chain LangChain normale
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

chain = ChatPromptTemplate.from_messages([
    ("human", "{question}")
]) | llm | StrOutputParser()

réponse = chain.invoke({"question": "C'est quoi un transformer ?"})
```

> [!tip] pipeline() pour prototyper, AutoModel pour contrôler
> `pipeline()` est parfait pour tester rapidement un modèle. Quand tu as besoin de contrôle fin (accès aux logits, batch custom, fine-tuning), utilise directement `AutoModel` + `AutoTokenizer`.

> [!warning] Premier chargement = téléchargement
> Le premier appel télécharge le modèle depuis le Hub et le met en cache (`~/.cache/huggingface/`). Prévoir l'espace disque nécessaire (Mistral 7B ≈ 14GB en float16, ≈ 4GB en 4-bit).
