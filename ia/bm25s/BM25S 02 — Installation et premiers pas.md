#ia #bm25s #installation #bases #pratique

## Installation et premiers pas

## Installation

```bash
# Recommandé — toutes les dépendances core
# (stemming, barre de progression, JSON, JIT numba)
pip install "bm25s[core]"

# Minimal — juste numpy et scipy
pip install bm25s

# Complet — tous les extras
pip install "bm25s[full]"

# Alternative simplifiée (depuis mars 2026)
pip install bm25          # wrapper avec API encore plus simple

# Stemming français (très recommandé)
pip install PyStemmer
```

## Pipeline complet en 4 étapes

```python
import bm25s

# ── Étape 1 : Définir le corpus ───────────────────────────
corpus = [
    "La politique de retour est de 30 jours après réception.",
    "La livraison gratuite s'applique dès 50 euros d'achat.",
    "La garantie couvre 2 ans pièces et main d'œuvre.",
    "La livraison express est disponible en 24h pour 9,90€.",
    "Les produits défectueux sont repris sans frais supplémentaires.",
    "Le remboursement est effectué sous 5 à 7 jours ouvrés.",
]

# ── Étape 2 : Tokeniser ───────────────────────────────────
# bm25s.tokenize() segmente le texte en tokens (mots)
corpus_tokenisé = bm25s.tokenize(corpus)

# ── Étape 3 : Indexer ─────────────────────────────────────
retriever = bm25s.BM25()
retriever.index(corpus_tokenisé)

# ── Étape 4 : Rechercher ──────────────────────────────────
query = "délai retour produit"
query_tokenisée = bm25s.tokenize(query)

résultats, scores = retriever.retrieve(
    query_tokenisée,
    corpus=corpus,   # passer le corpus pour récupérer les textes
    k=3              # top-3 résultats
)

for i in range(résultats.shape[1]):
    print(f"Score {scores[0, i]:.3f} | {résultats[0, i]}")

# → Score 2.41 | La politique de retour est de 30 jours après réception.
# → Score 1.23 | Le remboursement est effectué sous 5 à 7 jours ouvrés.
# → Score 0.87 | Les produits défectueux sont repris sans frais supplémentaires.
```

## Comprendre la sortie de retrieve()

```python
résultats, scores = retriever.retrieve(query_tokenisée, corpus=corpus, k=3)

# résultats : numpy array de shape (nb_queries, k)
#   résultats[0]    → résultats de la première (et unique) query
#   résultats[0, 0] → meilleur résultat (texte si corpus fourni, sinon index)
#   résultats[0, 1] → 2ème meilleur résultat

# scores : numpy array de shape (nb_queries, k)
#   scores[0, 0]    → score BM25 du meilleur résultat (plus élevé = plus pertinent)

print(résultats.shape)   # → (1, 3)
print(scores.shape)      # → (1, 3)
```

## Recherche multi-requêtes (batch)

```python
# Plusieurs requêtes en un seul appel
queries = [
    "délai retour produit",
    "livraison gratuite seuil",
    "garantie pièces"
]

queries_tokenisées = bm25s.tokenize(queries)

résultats, scores = retriever.retrieve(queries_tokenisées, corpus=corpus, k=2)

for q_idx, query in enumerate(queries):
    print(f"\nQuery : {query}")
    for i in range(résultats.shape[1]):
        print(f"  {scores[q_idx, i]:.2f} | {résultats[q_idx, i][:60]}")
```

## Récupérer les indices plutôt que les textes

```python
# Sans passer corpus → retourne les indices des documents
résultats_idx, scores = retriever.retrieve(query_tokenisée, k=3)

for i in range(résultats_idx.shape[1]):
    idx   = résultats_idx[0, i]
    score = scores[0, i]
    texte = corpus[idx]   # accès manuel au corpus
    print(f"Index {idx} | Score {score:.3f} | {texte[:60]}")
```

## Le tokenizer par défaut casse les identifiants avec tiret

`bm25s.tokenize()` utilise par défaut le motif `\b\w\w+\b`, qui ne considère pas le tiret comme un caractère de mot — un identifiant technique comme `--volumes-from` ou un code d'erreur `E-4042` est donc découpé en fragments séparés (`volumes`, `from`) plutôt que conservé comme un seul token exact.

> [!warning] Sur de la documentation technique, ce découpage fait perdre l'exactitude
> BM25 excelle précisément sur les termes exacts (identifiants, options de ligne de commande, codes d'erreur) — c'est son avantage sur la recherche dense, qui dilue ces termes rares dans le sens général de la phrase (voir [[RAG 08 — Hybrid RAG]]). Si le tokenizer par défaut fragmente ces identifiants, cet avantage disparaît silencieusement, sans erreur visible.

```python
import re

def splitter_technique(texte: str) -> list[str]:
    """Tokenise en conservant les identifiants avec tiret intacts."""
    return re.findall(r"[\w\-]+", texte.lower())

corpus_tokenisé = bm25s.tokenize(corpus, splitter=splitter_technique)
```

> [!tip] Le paramètre `splitter` accepte une regex ou une fonction
> `bm25s.tokenize(..., splitter=...)` accepte soit une chaîne interprétée comme motif regex, soit une fonction personnalisée — sur un corpus riche en identifiants techniques, tester explicitement qu'une requête contenant l'identifiant exact retrouve bien le bon document est le meilleur moyen de détecter ce piège avant qu'il n'affecte la production.

## Paramètres BM25 ajustables

```python
retriever = bm25s.BM25(
    k1=1.5,    # poids de la fréquence du terme (défaut: 1.5)
               # k1 élevé = terme fréquent très avantagé
               # k1 faible = saturation rapide de la fréquence

    b=0.75,    # normalisation par la longueur du document (défaut: 0.75)
               # b=1.0 = normalisation complète par longueur
               # b=0.0 = pas de normalisation

    method="bm25"   # "bm25" (Okapi), "bm25l", "bm25+"
)
```

> [!tip] Garder les paramètres par défaut
> `k1=1.5` et `b=0.75` sont les valeurs standards de la littérature scientifique. Ils fonctionnent bien dans la majorité des cas. N'ajuste que si tu as des données pour valider l'amélioration.

> [!warning] Tokenisation cohérente
> La même fonction de tokenisation doit être utilisée pour l'indexation ET pour les requêtes. Si tu utilises un stemmer à l'indexation, utilise le même stemmer pour les requêtes, sinon les scores seront incohérents.
