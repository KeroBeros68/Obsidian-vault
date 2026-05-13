#ia #rag #embeddings #vectordb #bases

## Embeddings et Vector Databases

Ces deux concepts sont le moteur de tout système RAG. Sans eux, la recherche sémantique est impossible.

## Les Embeddings — transformer du texte en nombres

Un embedding est la transformation d'un texte en **vecteur numérique** — une liste de centaines ou milliers de nombres.

```
"Procédure de retour produit"   → [0.23, -0.87, 0.41, 0.12, ...]
"Comment renvoyer un article"   → [0.21, -0.85, 0.43, 0.11, ...]  ← très proches !
"Recette de cuisine au chocolat" → [0.89,  0.12, -0.67, 0.55, ...] ← très différent
```

Deux phrases qui **veulent dire la même chose** ont des vecteurs **proches** dans l'espace mathématique — même si les mots sont totalement différents.

## Analogie

```
Imagine une carte géographique où les idées sont des villes.

"Retour produit" et "renvoyer un article" → à 2 km l'un de l'autre
"Retour produit" et "remboursement"       → à 10 km
"Retour produit" et "recette de cuisine"  → à 5 000 km
```

La "distance" entre deux vecteurs mesure leur similarité sémantique.

## Comment la similarité est calculée

La mesure la plus courante est la **similarité cosinus** — elle mesure l'angle entre deux vecteurs.

| Similarité | Valeur | Signification |
|---|---|---|
| 1.0 | Identique | Même sens exact |
| 0.8-0.9 | Très proche | Même sujet, mots différents |
| 0.5-0.7 | Lié | Sujet connexe |
| 0.0-0.3 | Éloigné | Sujets sans rapport |

## Modèles d'embedding populaires

| Modèle | Créateur | Points forts |
|---|---|---|
| `text-embedding-3-small` | OpenAI | Rapide, peu coûteux |
| `text-embedding-3-large` | OpenAI | Plus précis |
| `embed-multilingual-v3` | Cohere | Excellent multilingue dont français |
| `all-MiniLM-L6-v2` | Sentence Transformers | Gratuit, open-source |

> [!tip] Pour le français
> Privilégie un modèle multilingue comme celui de Cohere pour de meilleurs résultats en français.

## Les Vector Databases — stocker et chercher des vecteurs

Une base de données classique cherche par **mots-clés exacts**.
Une vector database cherche par **similarité sémantique**.

| Base classique | Vector Database |
|---|---|
| `SELECT * WHERE texte = "retour"` | Trouve tout ce qui est sémantiquement proche de "retour produit" |
| Cherche les mots exacts | Comprend le sens |
| SQL | API de recherche vectorielle |

## Les principales vector databases

| Outil | Type | Points forts |
|---|---|---|
| **Chroma** | Open-source, local | Parfait pour débuter, zéro config |
| **Pinecone** | Cloud managé | Simple, prêt pour la production |
| **Weaviate** | Open-source | Puissant, hybride vectoriel+BM25 |
| **Qdrant** | Open-source | Très performant, filtres avancés |
| **pgvector** | Extension PostgreSQL | Si tu as déjà une base Postgres |
| **FAISS** | Librairie (Meta) | Ultra-rapide, pour grands volumes |

> [!tip] Par où commencer
> Chroma pour apprendre et prototyper. Pinecone ou Qdrant pour un projet en production.

## Le chunking — découper les documents

Avant d'indexer, il faut découper les documents en morceaux (**chunks**). La taille impacte directement la qualité.

| Taille chunk | Avantages | Inconvénients |
|---|---|---|
| Petit (128-256 tokens) | Précis, peu de bruit | Peut perdre le contexte |
| Moyen (512-1024 tokens) | Bon équilibre | Standard recommandé |
| Grand (2048+ tokens) | Contexte riche | Moins précis, plus de bruit |

> [!warning] Le chunk size est crucial
> Trop petit = la réponse manque de contexte. Trop grand = des informations non pertinentes polluent la réponse. Le bon réglage dépend de tes documents — teste les deux extrêmes.
