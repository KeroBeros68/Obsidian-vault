#ia #rag #advanced #intermédiaire

## Advanced RAG

Le Naive RAG amélioré. Il ajoute des étapes intelligentes **avant** et **après** la recherche pour améliorer la qualité des résultats.

## Vue d'ensemble du pipeline

```
         PRE-RETRIEVAL              RETRIEVAL         POST-RETRIEVAL
┌──────────────────────────┐   ┌─────────────┐   ┌──────────────────┐
│ Query rewriting          │   │             │   │ Re-ranking        │
│ Query expansion          │──>│  Vector DB  │──>│ Compression       │
│ HyDE                     │   │             │   │ Filtrage          │
└──────────────────────────┘   └─────────────┘   └────────┬─────────┘
                                                           ↓
                                                       [LLM] → Réponse
```

## Pre-retrieval — améliorer la question

### Query Rewriting
Reformule la question pour mieux correspondre aux documents indexés.

```
Question originale : "ça marche pas mon compte"
Après rewriting    : "problème de connexion compte utilisateur, causes et solutions"
```

### Query Expansion
Génère plusieurs variantes de la question et lance plusieurs recherches en parallèle.

```
Question : "tarifs abonnement"
Variantes générées :
  → "prix des formules d'abonnement"
  → "coût mensuel annuel"
  → "grille tarifaire offres"
→ Les 3 recherches sont fusionnées
```

### HyDE (Hypothetical Document Embeddings)
Génère une réponse hypothétique à la question, puis cherche des documents similaires à cette réponse — souvent plus efficace que chercher la question directement.

```
Question → LLM génère une réponse fictive → on cherche des docs proches de cette réponse
```

> [!tip] HyDE contre-intuitif mais puissant
> Une "réponse imaginée" a souvent plus de mots en commun avec les vrais documents qu'une question courte.

## Post-retrieval — améliorer les résultats

### Re-ranking
Un second modèle reclasse les chunks par pertinence réelle après la recherche vectorielle.

```
Recherche vectorielle retourne : [chunk A, chunk B, chunk C, chunk D, chunk E]
Re-ranker analyse et reclasse  : [chunk C, chunk A, chunk E, chunk B, chunk D]
On garde seulement les top 3   : [chunk C, chunk A, chunk E]
```

Modèles de re-ranking populaires : `cross-encoder/ms-marco-MiniLM`, `bge-reranker-v2-m3`, Cohere Rerank.

> [!info] Pourquoi ça marche, et comment l'implémenter
> Le re-ranking repose sur un **cross-encoder**, structurellement différent du bi-encoder utilisé pour la recherche — voir [[RAG 10 — Re-ranking avec un cross-encoder]] pour la distinction, l'implémentation complète et le critère pour décider si le coût en latence se justifie.

### Compression
Supprime les parties non pertinentes dans les chunks récupérés avant de les envoyer au LLM. Réduit le bruit et les coûts.

```
Chunk original (500 tokens) → Compresseur → Passage pertinent (80 tokens)
```

## Quand utiliser chaque technique

| Problème observé | Solution |
|---|---|
| Questions mal formulées par les utilisateurs | Query Rewriting |
| Questions sur plusieurs aspects à la fois | Query Expansion |
| Mauvais résultats malgré bonne question | HyDE |
| Chunks récupérés peu pertinents | Re-ranking |
| Réponses polluées par trop d'informations | Compression |

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Bien meilleure qualité que Naive RAG | Plus complexe à mettre en place |
| Résistant aux questions mal formulées | Latence plus élevée (plusieurs étapes) |
| Standard en production aujourd'hui | Coût légèrement supérieur |

> [!warning] Ne pas tout activer d'un coup
> Ajoute les techniques une par une. Mesure l'amélioration à chaque étape. Certaines techniques n'apportent rien sur ton cas d'usage spécifique.
