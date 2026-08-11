#ia #transformers #résumé #traduction #seq2seq #pratique

## Résumé et traduction

Les modèles seq2seq (BART, T5, mBART, NLLB) sont spécialisés dans les tâches de transformation de texte : résumé, traduction, reformulation, question-génération.

## Résumé automatique

```python
from transformers import pipeline

# ── Résumé anglais (BART) ─────────────────────────────────
pipe_résumé = pipeline(
    "summarization",
    model="facebook/bart-large-cnn",
    device=0
)

texte_long = """
Retrieval-Augmented Generation (RAG) is an AI framework that enhances
large language models by allowing them to retrieve relevant information
from external knowledge bases before generating responses. Unlike
traditional LLMs that rely solely on their training data, RAG systems
can access up-to-date information and domain-specific documents. The
retrieval component finds relevant passages using vector similarity
search, while the generation component uses these passages as context
to produce accurate, grounded responses. This approach reduces
hallucinations and enables the model to cite sources.
"""

résumé = pipe_résumé(
    texte_long,
    max_length=100,   # longueur max du résumé (tokens)
    min_length=30,    # longueur min
    do_sample=False   # greedy pour la cohérence
)
print(résumé[0]["summary_text"])

# ── Résumé français (mBARThez) ────────────────────────────
pipe_résumé_fr = pipeline(
    "summarization",
    model="lincoln/mbart-mlsum-automatic-summarization"
)

texte_fr = """
Le Retrieval-Augmented Generation (RAG) est une technique qui combine
la recherche dans une base documentaire avec la génération de texte
par un LLM. Au lieu de répondre uniquement depuis sa mémoire,
le modèle récupère d'abord les passages les plus pertinents dans
une base vectorielle, puis les utilise comme contexte pour générer
une réponse précise et sourcée. Cette approche réduit les
hallucinations et permet de maintenir des connaissances à jour.
"""

résumé_fr = pipe_résumé_fr(texte_fr, max_length=80, min_length=20)
print(résumé_fr[0]["summary_text"])
```

## Résumé avec T5

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM
import torch

tokenizer = AutoTokenizer.from_pretrained("t5-base")
model     = AutoModelForSeq2SeqLM.from_pretrained("t5-base")

# T5 nécessite un préfixe de tâche
texte = "summarize: " + texte_long

inputs = tokenizer(texte, return_tensors="pt", max_length=1024, truncation=True)

with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_length=150,
        min_length=40,
        num_beams=4,
        no_repeat_ngram_size=3,
        early_stopping=True
    )

résumé = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(résumé)
```

## Traduction avec Helsinki-NLP (OPUS-MT)

```python
from transformers import pipeline

# ── FR → EN ───────────────────────────────────────────────
pipe_trad = pipeline("translation_fr_to_en", model="Helsinki-NLP/opus-mt-fr-en")
résultat  = pipe_trad("Bonjour, comment puis-je vous aider aujourd'hui ?")
print(résultat[0]["translation_text"])
# → "Hello, how can I help you today?"

# ── EN → FR ───────────────────────────────────────────────
pipe_trad = pipeline("translation_en_to_fr", model="Helsinki-NLP/opus-mt-en-fr")
résultat  = pipe_trad("The model successfully retrieved the relevant documents.")
print(résultat[0]["translation_text"])

# ── Autres paires de langues ──────────────────────────────
# Helsinki-NLP/opus-mt-{src}-{tgt} où src et tgt sont des codes ISO
# Exemples : de-en, es-en, it-fr, zh-en, ja-en, ar-en...
```

## Traduction multilingue avec NLLB

NLLB (No Language Left Behind, Meta) supporte 200 langues en un seul modèle.

```python
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer

modèle_id = "facebook/nllb-200-distilled-600M"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForSeq2SeqLM.from_pretrained(modèle_id)

def traduire(texte: str, lang_source: str, lang_cible: str) -> str:
    """
    Traduit un texte de lang_source vers lang_cible.
    Codes de langue NLLB : fra_Latn (français), eng_Latn (anglais),
    deu_Latn (allemand), spa_Latn (espagnol), zho_Hans (chinois simplifié)...
    """
    tokenizer.src_lang = lang_source

    inputs = tokenizer(texte, return_tensors="pt", truncation=True, max_length=512)

    forced_bos_id = tokenizer.lang_code_to_id[lang_cible]
    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            forced_bos_token_id=forced_bos_id,
            max_new_tokens=512
        )

    return tokenizer.decode(outputs[0], skip_special_tokens=True)

# Exemples
print(traduire("Bonjour le monde !", "fra_Latn", "eng_Latn"))
# → "Hello world!"

print(traduire("The cat sat on the mat.", "eng_Latn", "fra_Latn"))
# → "Le chat était assis sur le tapis."

print(traduire("Hola mundo", "spa_Latn", "fra_Latn"))
# → "Bonjour monde"
```

## Pipeline de résumé + traduction (chain)

```python
from transformers import pipeline

# Résumer en français puis traduire en anglais
pipe_résumé_fr = pipeline("summarization", model="lincoln/mbart-mlsum-automatic-summarization")
pipe_trad_fr_en = pipeline("translation_fr_to_en", model="Helsinki-NLP/opus-mt-fr-en")

texte_fr = "Long texte en français à résumer et traduire..."

# Étape 1 : résumer
résumé = pipe_résumé_fr(texte_fr, max_length=100)[0]["summary_text"]
print(f"Résumé FR : {résumé}")

# Étape 2 : traduire
traduction = pipe_trad_fr_en(résumé)[0]["translation_text"]
print(f"Traduction EN : {traduction}")
```

## Modèles seq2seq — choisir le bon

| Modèle | Tâche | Langue | Taille |
|---|---|---|---|
| `facebook/bart-large-cnn` | Résumé | Anglais | 400M |
| `lincoln/mbart-mlsum-automatic-summarization` | Résumé | Français | 600M |
| `google/pegasus-xsum` | Résumé abstractif | Anglais | 568M |
| `Helsinki-NLP/opus-mt-fr-en` | Traduction FR→EN | FR/EN | 77M |
| `facebook/nllb-200-distilled-600M` | Traduction | 200 langues | 600M |
| `facebook/mbart-large-50-many-to-many-mmt` | Traduction | 50 langues | 611M |
| `t5-base` | Multi-tâche | Anglais | 250M |
| `google/flan-t5-large` | Instruction-tuned | Anglais | 780M |

> [!tip] NLLB pour le multilingue
> Si ton application doit gérer plus de 2 langues, `facebook/nllb-200-distilled-600M` est le meilleur choix — un seul modèle pour 200 langues, qualité proche des modèles dédiés, et il tient sur un GPU standard.

> [!warning] Longueur d'entrée des modèles seq2seq
> La plupart des modèles seq2seq ont une limite d'entrée de 512 à 1024 tokens. Pour les longs documents, découpe d'abord en sections et résume chaque section, puis résume les résumés (résumé hiérarchique).
