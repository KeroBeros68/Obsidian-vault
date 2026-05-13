#ia #rag #hybrid #intermédiaire

## Hybrid RAG

Combine **deux types de recherche complémentaires** : la recherche vectorielle (sémantique) et la recherche par mots-clés (BM25). C'est aujourd'hui l'approche la plus efficace en production.

## Pourquoi les deux approches seules sont insuffisantes

### Recherche vectorielle seule
- ✅ Comprend le sens, gère les synonymes
- ❌ Peut rater des termes techniques très spécifiques

```
"code erreur P0300" → la recherche vectorielle peut trouver
"problèmes moteur" au lieu du document exact sur "P0300"
```

### Recherche BM25 (mots-clés) seule
- ✅ Trouve les termes exacts, parfait pour codes, noms propres
- ❌ Ne comprend pas le sens, rigide

```
"voiture ne démarre pas" → ne trouve pas le document
qui parle de "panne de démarrage" avec des mots différents
```

## Le meilleur des deux mondes

```
Question utilisateur
        ↓
   ┌────┴────┐
   ↓         ↓
Recherche  Recherche
Vectorielle  BM25
(sémantique) (mots-clés)
   ↓         ↓
   └────┬────┘
        ↓
   [Fusion des résultats]
        ↓
   [Re-ranking optionnel]
        ↓
      [LLM]
        ↓
     Réponse
```

## L'algorithme de fusion — RRF

Le **Reciprocal Rank Fusion (RRF)** est la méthode standard pour combiner les deux listes de résultats.

```
Résultats vectoriels : [Doc A (rank 1), Doc C (rank 2), Doc B (rank 3)]
Résultats BM25       : [Doc B (rank 1), Doc A (rank 2), Doc D (rank 3)]

Score RRF(Doc A) = 1/(60+1) + 1/(60+2) = 0.0164 + 0.0161 = 0.0325
Score RRF(Doc B) = 1/(60+3) + 1/(60+1) = 0.0159 + 0.0164 = 0.0323
Score RRF(Doc C) = 1/(60+2) + 0        = 0.0161

Résultat fusionné : [Doc A, Doc B, Doc C, Doc D]
```

> [!info] Pourquoi 60 dans la formule ?
> C'est une constante empirique qui donne un bon équilibre. Elle évite qu'un rang 1 écrase complètement les autres résultats.

## Cas d'usage typiques

| Type de question | Méthode qui gagne |
|---|---|
| "Comment fonctionne le remboursement ?" | Vectorielle (sens) |
| "Que dit l'article 7.2.1 du contrat ?" | BM25 (terme exact) |
| "code erreur ERR_CONNECTION_RESET" | BM25 (code exact) |
| "mon application est lente" | Vectorielle (sens) |
| "procédure de retour pour les produits défectueux" | Les deux |

## Implémentation

Weaviate supporte nativement le Hybrid RAG. LlamaIndex et LangChain ont aussi des connecteurs hybrides.

```python
# Pseudo-code LlamaIndex
retriever = QueryFusionRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    similarity_top_k=5,
    num_queries=3,
    mode="reciprocal_rerank"
)
```

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Meilleure couverture que chaque méthode seule | Deux index à maintenir |
| Très efficace en production | Légèrement plus complexe |
| Robuste à tous types de questions | Latence un peu plus élevée |
| Standard recommandé pour les apps réelles | — |

> [!tip] Recommandation
> Si tu construis un RAG pour la production et que tu ne sais pas quel type choisir, commence par le Hybrid RAG. C'est le meilleur rapport qualité/complexité pour la majorité des cas d'usage.
