#ia #transformers #tokenisation #bpe #bases #intermédiaire

## Tokenisation en détail

La tokenisation est l'étape qui transforme du texte brut en séquences de nombres que le modèle peut traiter. Comprendre ce mécanisme explique beaucoup de comportements des LLM.

## Pourquoi tokeniser ?

```
Les LLM ne comprennent pas le texte directement.
Ils opèrent sur des séquences de nombres entiers (token IDs).

"Bonjour" → [661, 1785] → modèle → [1234, 5678, ...] → "Hello"
```

## Les algorithmes de tokenisation

### BPE — Byte Pair Encoding (GPT, Mistral, LLaMA)

Apprend à fusionner les paires de bytes les plus fréquentes.

```
Corpus : "aababaaab"
Init   : a, a, b, a, b, a, a, a, b   (bytes individuels)
Étape 1: "aa" est la paire la plus fréquente → fusionne en "aa"
Étape 2: "ab" est la plus fréquente → fusionne en "ab"
...
Résultat: un vocabulaire de sous-mots fréquents
```

### WordPiece (BERT, CamemBERT)

Similaire à BPE mais maximise la probabilité du corpus d'entraînement.
Les sous-mots commencent par `##` pour signaler une continuation.

```
"tokenisation" → ["token", "##isa", "##tion"]
"joueur"        → ["joueur"]
"joueurs"       → ["joueur", "##s"]
```

### SentencePiece (T5, ALBERT, mT5)

Traite le texte brut sans pré-tokenisation — supporte toutes les langues.
Les tokens commencent par `▁` (espace) pour marquer les débuts de mots.

```
"Bonjour monde" → ["▁Bon", "jour", "▁monde"]
```

## Explorer la tokenisation d'un modèle

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")

# Voir la taille du vocabulaire
print(tokenizer.vocab_size)   # → 32000 (Mistral)
# GPT-4 : ~100k, LLaMA 3 : ~128k, BERT : 30k

# Tokeniser et voir les tokens
texte = "Le RAG combine la recherche vectorielle et la génération."
tokens = tokenizer.tokenize(texte)
print(tokens)
# → ['▁Le', '▁R', 'AG', '▁combine', '▁la', '▁recherche',
#    '▁vectori', 'elle', '▁et', '▁la', '▁génération', '.']

# IDs numériques
ids = tokenizer.encode(texte)
print(ids)   # → [1, 1978, 399, 1926, 16810, ...]

# Compter les tokens d'un texte
nb_tokens = len(tokenizer.encode(texte))
print(f"{nb_tokens} tokens pour {len(texte)} caractères")
# → Règle approx : 1 token ≈ 4 chars en anglais, ≈ 3 chars en français
```

## Les cas surprenants de la tokenisation

```python
tokenizer = AutoTokenizer.from_pretrained("gpt2")

# Les nombres sont souvent découpés
print(tokenizer.tokenize("1234567890"))
# → ['12', '345', '678', '90']  ← pas de token par chiffre

# Les caractères spéciaux sont variables
print(tokenizer.tokenize("C'est l'IA !"))
# → ["C", "'", "est", " l", "'", "IA", " !"]

# Les mots rares sont découpés en sous-mots
print(tokenizer.tokenize("transformerization"))
# → ["transform", "erization"]  ← sous-mots plus fréquents

# La casse crée des tokens différents
print(tokenizer.tokenize("Bonjour bonjour"))
# → ["Bon", "jour", " bon", "jour"]  ← majuscule = token différent

# Les langues non-latines consomment plus de tokens
print(len(tokenizer.encode("Hello world")))  # → 2 tokens
print(len(tokenizer.encode("你好世界")))      # → beaucoup plus
```

## Compter les tokens pour estimer les coûts

```python
from transformers import AutoTokenizer

def compter_tokens(texte: str, modèle: str = "mistralai/Mistral-7B-Instruct-v0.3") -> int:
    tokenizer = AutoTokenizer.from_pretrained(modèle)
    return len(tokenizer.encode(texte))

# Estimer le coût d'un prompt
system_prompt = "Tu es un assistant expert en droit français..."
question = "Quelle est la durée de prescription pour un litige commercial ?"
contexte = "Document RAG récupéré de 2000 mots..."

total = compter_tokens(system_prompt + question + contexte)
print(f"Tokens input : {total}")

# Règle d'or pour estimer sans tokenizer :
# Anglais : 1 token ≈ 4 caractères ≈ 0.75 mots
# Français : 1 token ≈ 3 caractères ≈ 0.65 mots
# Code    : 1 token ≈ 3 caractères (plus varié)
```

## Les limites de la context window

```python
# Chaque modèle a une limite de tokens par appel
limites = {
    "Mistral-7B-Instruct-v0.3": 32_768,
    "LLaMA-3.1-8B-Instruct":    128_000,
    "Qwen3-8B":                  128_000,
    "GPT-4o":                    128_000,
    "Claude Sonnet":             200_000,
}

# Vérifier si un texte dépasse la limite
def vérifie_longueur(texte: str, modèle_id: str, limite: int) -> bool:
    tokenizer = AutoTokenizer.from_pretrained(modèle_id)
    nb = len(tokenizer.encode(texte))
    if nb > limite:
        print(f"⚠️ {nb} tokens > limite de {limite}")
        return False
    print(f"✅ {nb} / {limite} tokens")
    return True
```

## Tokenizer rapide vs lent

```python
# Par défaut, Transformers utilise le tokenizer rapide (Rust, via tokenizers)
tokenizer = AutoTokenizer.from_pretrained("mistralai/Mistral-7B-Instruct-v0.3")
print(tokenizer.is_fast)   # → True

# Le tokenizer rapide expose des informations supplémentaires
encoding = tokenizer("Bonjour monde", return_offsets_mapping=True)
print(encoding["offset_mapping"])
# → [(0,0), (0,7), (7,8), (8,13)]
# Positions de chaque token dans le texte original → utile pour le NER
```

> [!tip] Règle des 4 caractères
> En anglais, 1 token ≈ 4 caractères. En français, ≈ 3-3.5 (les mots sont plus longs). Pour du code : très variable selon le langage. Cette approximation suffit pour estimer les coûts sans appeler le tokenizer.

> [!warning] Le même texte = tokens différents selon le modèle
> "Hello world" peut donner 2 tokens avec Mistral et 3 avec BERT. Ne jamais supposer que la tokenisation est universelle — chaque modèle a son propre vocabulaire.
