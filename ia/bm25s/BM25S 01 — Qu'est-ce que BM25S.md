#ia #bm25s #bases #définition #recherche

## Qu'est-ce que BM25S ?

BM25S est une implémentation Python de l'algorithme BM25 (Okapi BM25) qui atteint jusqu'à **500× le speedup** de `rank-bm25`, la librairie la plus populaire, grâce à des matrices creuses SciPy et un calcul "eager" des scores.

## BM25 — le rappel

BM25 est un algorithme de recherche par **mots-clés** (lexicale). Il note chaque document selon deux facteurs :

```
TF  (Term Frequency)     : combien de fois le mot apparaît dans le document
IDF (Inverse Doc Freq)   : à quel point le mot est rare dans le corpus

Score BM25(doc, query) = Σ IDF(mot) × TF normalisé(mot, doc)
```

> C'est ce qui propulse Elasticsearch, Lucene, et la plupart des moteurs de recherche industriels.

## Le problème de rank-bm25

```
rank-bm25 (librairie historique) :
  ✅ Simple à utiliser
  ❌ Calcul des scores à la volée pour chaque requête
  ❌ Lent sur les grands corpus (> 10k documents)
  ❌ Gourmand en mémoire
  ❌ Pas de persistance de l'index

bm25s :
  ✅ Pré-calcule tous les scores au moment de l'indexation (eager scoring)
  ✅ Stocke dans des matrices sparse SciPy → accès O(1) au moment de la requête
  ✅ Jusqu'à 500× plus rapide que rank-bm25
  ✅ Index persistable sur disque
  ✅ Intégration Hugging Face Hub
```

## L'innovation clé — le scoring "eager"

```
rank-bm25 (lazy) :
  Requête arrive → calcule le score pour chaque document → retourne top-k
  → O(n × longueur_query) pour chaque requête

bm25s (eager) :
  Indexation → calcule et stocke les scores pour TOUS les tokens
  Requête arrive → lookup dans la matrice sparse → somme → retourne top-k
  → O(longueur_query) pour chaque requête après indexation
```

## Benchmarks (corpus BEIR, single-thread)

```
Librairie       Speedup vs rank-bm25
─────────────────────────────────────
rank-bm25       1× (référence)
bm25s           100× à 500× selon le corpus
Elasticsearch   comparable à bm25s (mais nécessite Java + serveur)
```

## Positionnement dans l'écosystème

```
Recherche lexicale pure :
  rank-bm25   → petit corpus, intégration LangChain native
  bm25s       → tout nouveau projet, corpus moyen à large
  Elasticsearch → corpus > 1M docs, infra déjà en place

Recherche hybride (recommandé en prod) :
  bm25s + vectorstore → meilleur des deux mondes
  ← c'est l'usage principal de bm25s dans le contexte RAG
```

## Versions disponibles de l'algorithme

BM25S implémente plusieurs variantes de BM25 :

| Variante | Description | Usage |
|---|---|---|
| `BM25Okapi` | Version standard | Défaut, bon pour tout |
| `BM25L` | Pénalise moins les longs documents | Documents de longueur variable |
| `BM25Plus` | Score minimal garanti pour les termes présents | Évite le biais contre les courts docs |

```python
import bm25s

# Choisir la variante
retriever = bm25s.BM25(method="bm25")      # Okapi (défaut)
retriever = bm25s.BM25(method="bm25l")     # BM25L
retriever = bm25s.BM25(method="bm25+")     # BM25Plus
```

> [!tip] BM25S vs rank-bm25 dans LangChain
> Le `BM25Retriever` natif de LangChain utilise `rank_bm25` sous le capot. Pour utiliser bm25s dans LangChain, il faut créer un retriever custom — voir [[BM25S 05 — Intégration LangChain]].

> [!info] Nouveau : pip install bm25
> Depuis mars 2026, `pip install bm25` installe bm25s sous le capot avec une API encore plus simple et une CLI intégrée. Les deux packages sont maintenus par le même auteur (Xing Han Lù).
