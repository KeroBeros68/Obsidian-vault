#ia #transformers #classification #ner #nlp #pratique

## Classification et NER

BERT et ses variantes (RoBERTa, CamemBERT, DistilBERT) sont optimisés pour les tâches de compréhension : classification de texte, analyse de sentiment, NER (reconnaissance d'entités nommées).

## Classification de texte — analyse de sentiment

```python
from transformers import pipeline

# ── Sentiment anglais ─────────────────────────────────────
pipe = pipeline(
    "text-classification",
    model="distilbert-base-uncased-finetuned-sst-2-english",
    device=0   # GPU si disponible
)

résultats = pipe([
    "This product is absolutely amazing!",
    "Terrible quality, completely disappointed.",
    "It's okay, nothing special."
])

for r in résultats:
    print(f"{r['label']:10} | {r['score']:.3f}")
# → POSITIVE   | 0.9998
# → NEGATIVE   | 0.9994
# → NEGATIVE   | 0.6823  (ambigu)

# ── Sentiment français ────────────────────────────────────
pipe_fr = pipeline(
    "text-classification",
    model="cmarkea/distilcamembert-base-sentiment"
)
résultat = pipe_fr("Ce produit est excellent, je recommande vivement !")
print(résultat)   # → [{"label": "5 stars", "score": 0.94}]
```

## Classification zero-shot — sans fine-tuning

```python
from transformers import pipeline

pipe = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")

texte = "Je veux retourner mon article car il est défectueux."
labels = ["retour produit", "livraison", "garantie", "paiement", "autre"]

résultat = pipe(texte, candidate_labels=labels, multi_label=False)

for label, score in zip(résultat["labels"], résultat["scores"]):
    print(f"{label:20} : {score:.3f}")
# → retour produit      : 0.923
# → garantie            : 0.054
# → autre               : 0.013
# → livraison           : 0.007
# → paiement            : 0.003

# Multi-label (une phrase peut appartenir à plusieurs catégories)
résultat_multi = pipe(
    "Le colis est arrivé endommagé et je veux être remboursé.",
    candidate_labels=["livraison", "retour", "remboursement"],
    multi_label=True
)
```

## NER — Reconnaissance d'entités nommées

```python
from transformers import pipeline

# ── NER anglais ───────────────────────────────────────────
pipe_ner = pipeline(
    "ner",
    model="dslim/bert-base-NER",
    aggregation_strategy="simple"   # fusionne les sous-tokens d'une entité
)

texte = "Elon Musk founded Tesla in Palo Alto, California in 2003."
entités = pipe_ner(texte)

for e in entités:
    print(f"{e['word']:20} → {e['entity_group']:5} ({e['score']:.2f})")
# → Elon Musk           → PER   (0.99)
# → Tesla               → ORG   (0.99)
# → Palo Alto           → LOC   (0.98)
# → California          → LOC   (0.97)
# → 2003                → MISC  (0.72)

# ── NER français ─────────────────────────────────────────
pipe_ner_fr = pipeline(
    "ner",
    model="Jean-Baptiste/camembert-ner",
    aggregation_strategy="simple"
)

texte_fr = "Emmanuel Macron a rencontré Angela Merkel à Paris en 2021."
entités_fr = pipe_ner_fr(texte_fr)
for e in entités_fr:
    print(f"{e['word']:25} → {e['entity_group']}")
# → Emmanuel Macron      → PER
# → Angela Merkel        → PER
# → Paris                → LOC
```

## NER avec positions dans le texte

```python
from transformers import AutoTokenizer, AutoModelForTokenClassification
import torch

modèle_id = "dslim/bert-base-NER"
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForTokenClassification.from_pretrained(modèle_id)

texte = "Apple was founded by Steve Jobs in Cupertino."
inputs = tokenizer(texte, return_tensors="pt", return_offsets_mapping=True)
offset_mapping = inputs.pop("offset_mapping")   # garder de côté

with torch.no_grad():
    logits = model(**inputs).logits

# Décoder les prédictions
prédictions = logits.argmax(-1)[0]
labels = [model.config.id2label[p.item()] for p in prédictions]

tokens = tokenizer.convert_ids_to_tokens(inputs["input_ids"][0])
offsets = offset_mapping[0].tolist()

for token, label, (start, end) in zip(tokens, labels, offsets):
    if label != "O" and token not in ["[CLS]", "[SEP]"]:
        print(f"'{texte[start:end]}' → {label} (positions {start}-{end})")
# → 'Apple' → B-ORG (positions 0-5)
# → 'Steve' → B-PER (positions 22-27)
# → 'Jobs'  → I-PER (positions 28-32)
# → 'Cupertino' → B-LOC (positions 36-45)
```

## Pipeline de classification personnalisé

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch
import torch.nn.functional as F

# Modèle fine-tuné pour la classification de tickets support
modèle_id = "mon-org/ticket-classifier"  # modèle custom
tokenizer = AutoTokenizer.from_pretrained(modèle_id)
model     = AutoModelForSequenceClassification.from_pretrained(modèle_id)
model.eval()

def classifier_ticket(texte: str) -> dict:
    inputs = tokenizer(texte, return_tensors="pt", truncation=True, max_length=512)
    with torch.no_grad():
        logits = model(**inputs).logits
    probs  = F.softmax(logits, dim=-1)[0]
    labels = model.config.id2label
    return {labels[i]: float(probs[i]) for i in range(len(probs))}

résultat = classifier_ticket("Mon colis n'est pas arrivé après 5 jours.")
print(max(résultat, key=résultat.get))   # → "livraison"
```

## Catégories d'entités standards

```
NER classique (CoNLL-2003) :
  PER   : personnes
  ORG   : organisations
  LOC   : lieux
  MISC  : divers

NER étendu (OntoNotes) :
  PERSON, ORG, GPE (lieux politiques), DATE, TIME,
  MONEY, PERCENT, CARDINAL, ORDINAL, LAW...

Format BIO :
  B-PER : début d'entité personne
  I-PER : intérieur d'entité personne
  O     : pas d'entité
```

> [!tip] aggregation_strategy="simple"
> Toujours utiliser `aggregation_strategy="simple"` dans le pipeline NER. Sans ça, tu obtiens un token par sous-mot (`"Ste"`, `"##ve"`, `"Jo"`, `"##bs"`) au lieu de l'entité complète (`"Steve Jobs"`).

> [!info] CamemBERT pour le français
> CamemBERT est le modèle de référence pour le NLP français — entraîné sur un large corpus francophone. Préfère `Jean-Baptiste/camembert-ner` ou `cmarkea/distilcamembert-*` aux modèles anglais pour les textes français.
