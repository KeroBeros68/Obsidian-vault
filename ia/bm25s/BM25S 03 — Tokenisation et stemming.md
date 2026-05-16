#ia #bm25s #tokenisation #stemming #nlp #pratique

## Tokenisation et stemming

La qualité de la tokenisation impacte directement la qualité de la recherche. BM25S fournit un tokenizer intégré extensible, avec support du stemming et des stopwords.

## bm25s.tokenize() — le tokenizer intégré

```python
import bm25s

textes = [
    "La politique de retour est de 30 jours.",
    "Les produits défectueux sont repris sans frais."
]

# Tokenisation basique
tokens = bm25s.tokenize(textes)
print(tokens)
# → [["politique", "retour", "30", "jours"],
#    ["produits", "défectueux", "repris", "frais"]]
# Les mots vides ("la", "de", "est", "les", "sont", "sans") sont retirés par défaut
```

## Stopwords — supprimer les mots non informatifs

```python
# Stopwords intégrés par langue
tokens_fr = bm25s.tokenize(textes, stopwords="french")   # français
tokens_en = bm25s.tokenize(textes, stopwords="english")  # anglais

# Stopwords personnalisés
mes_stopwords = ["acme", "corp", "veuillez", "merci"]
tokens_custom = bm25s.tokenize(textes, stopwords=mes_stopwords)

# Sans stopwords (déconseillé — bruit)
tokens_bruts = bm25s.tokenize(textes, stopwords=None)
```

## Stemming — réduire les mots à leur racine

Le stemming améliore significativement le recall en faisant correspondre les variantes d'un même mot.

```
Sans stemming :
  "retour" ≠ "retours" ≠ "retourner" → 3 tokens différents
  Requête "retourner" ne trouve pas doc avec "retour"

Avec stemming :
  "retour" → "retour"
  "retours" → "retour"
  "retourner" → "retour"
  → Tous traités comme le même token → meilleur recall
```

```python
import bm25s
import Stemmer   # pip install PyStemmer

# Stemmers disponibles
stemmer_fr = Stemmer.Stemmer("french")
stemmer_en = Stemmer.Stemmer("english")
stemmer_es = Stemmer.Stemmer("spanish")
# Aussi : german, italian, portuguese, dutch, swedish, norwegian, danish...

corpus = [
    "La politique de retour est de 30 jours.",
    "Les retours sont acceptés sous conditions strictes.",
    "Vous pouvez retourner votre article défectueux."
]

# Tokenisation AVEC stemming + stopwords français
corpus_tokenisé = bm25s.tokenize(
    corpus,
    stemmer=stemmer_fr,
    stopwords="french"
)

retriever = bm25s.BM25()
retriever.index(corpus_tokenisé)

# Requête avec le MÊME stemmer
query = "retourner un produit endommagé"
query_tokenisée = bm25s.tokenize(query, stemmer=stemmer_fr, stopwords="french")

résultats, scores = retriever.retrieve(query_tokenisée, corpus=corpus, k=3)

# → Trouve les 3 documents car "retourner", "retour", "retours"
#   ont tous la même racine après stemming
```

## Tokenizer personnalisé

```python
import re

def tokenizer_personnalisé(texte: str) -> list[str]:
    """Tokenizer custom : minuscules + split sur non-alphanumériques."""
    texte = texte.lower()
    tokens = re.split(r'[^a-zàâéèêëîïôùûüç]+', texte)
    # Filtrer les tokens vides et trop courts
    return [t for t in tokens if len(t) > 2]

# Utiliser le tokenizer custom
corpus_tokenisé = bm25s.tokenize(
    corpus,
    tokenizer=tokenizer_personnalisé
)
```

## Pipeline complet avec stemming pour le français

```python
import bm25s
import Stemmer

def créer_retriever_fr(textes: list[str]) -> tuple:
    """Crée un retriever BM25S optimisé pour le français."""
    stemmer = Stemmer.Stemmer("french")

    corpus_tokenisé = bm25s.tokenize(
        textes,
        stemmer=stemmer,
        stopwords="french"
    )

    retriever = bm25s.BM25()
    retriever.index(corpus_tokenisé)

    return retriever, stemmer

def rechercher(retriever, stemmer, query: str, corpus: list[str], k: int = 4):
    """Recherche avec le même stemmer que l'indexation."""
    query_tokenisée = bm25s.tokenize(
        query,
        stemmer=stemmer,
        stopwords="french"
    )
    résultats, scores = retriever.retrieve(query_tokenisée, corpus=corpus, k=k)
    return [(résultats[0, i], float(scores[0, i])) for i in range(résultats.shape[1])]

# Utilisation
corpus = [
    "La politique de retour est de 30 jours.",
    "Livraison gratuite dès 50 euros.",
    "Garantie 2 ans pièces et main d'œuvre.",
]

retriever, stemmer = créer_retriever_fr(corpus)
résultats = rechercher(retriever, stemmer, "retourner un article", corpus)
for texte, score in résultats:
    print(f"{score:.3f} | {texte}")
```

## Accélération JIT avec Numba

Pour les très grands corpus, Numba compile le code Python en instructions natives.

```bash
pip install numba
```

```python
# Activer le backend numba (2× supplémentaire sur grands corpus)
retriever = bm25s.BM25(backend="numba")
retriever.index(corpus_tokenisé)
# → Premier appel = compilation JIT (quelques secondes)
# → Appels suivants = vitesse maximale
```

> [!tip] Stemming toujours recommandé en français
> Le français a beaucoup d'inflexions (genre, nombre, conjugaisons). Sans stemming, "livrée", "livraison", "livrer" sont 3 tokens différents. Avec stemming, ils convergent vers la même racine → bien meilleur recall.

> [!warning] Même tokenizer partout
> La règle d'or : **la fonction de tokenisation utilisée à l'indexation doit être identique à celle utilisée pour les requêtes**. Un stemmer à l'indexation sans stemmer à la requête = scores incohérents et mauvais résultats.
