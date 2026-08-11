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

La mesure la plus courante est la **similarité cosinus** — elle mesure l'angle entre deux vecteurs, indépendamment de leur longueur.

| Similarité | Valeur | Signification |
|---|---|---|
| 1.0 | Identique | Même sens exact |
| 0.8-0.9 | Très proche | Même sujet, mots différents |
| 0.5-0.7 | Lié | Sujet connexe |
| 0.0-0.3 | Éloigné | Sujets sans rapport |

Le calcul lui-même est de l'arithmétique pure — produit scalaire divisé par le produit des normes — sans aucun modèle impliqué à cette étape, ce qui le rend déterministe et testable isolément :

```python
import math

def similarite_cosinus(a: list[float], b: list[float]) -> float:
    """Mesure la proximité de deux vecteurs : 1 identique, 0 sans rapport."""
    produit = sum(x * y for x, y in zip(a, b))
    norme_a = math.sqrt(sum(x * x for x in a))
    norme_b = math.sqrt(sum(y * y for y in b))
    if norme_a == 0 or norme_b == 0:
        return 0.0
    return produit / (norme_a * norme_b)
```

> [!warning] Toujours le même modèle des deux côtés
> Les vecteurs de deux modèles d'embedding différents ne sont pas comparables — ils vivent dans des espaces mathématiques différents. La question posée par l'utilisateur et les documents indexés doivent être vectorisés par exactement le **même** modèle, à l'indexation comme à la recherche. Changer de modèle d'embedding impose de **réindexer tout le corpus** : conserver les anciens vecteurs mélangés aux nouveaux produit des scores de similarité incohérents, sans erreur visible pour le signaler.

> [!info] La dimension du vecteur est fixée par le modèle
> Chaque modèle produit des vecteurs d'une dimension fixe (768 pour `nomic-embed-text`, par exemple), quelle que soit la longueur du texte d'entrée. La base vectorielle doit être configurée avec exactement cette dimension — un changement de modèle sans réindexation complète provoque une erreur de dimension à l'écriture.

## Modèles d'embedding populaires

| Modèle | Créateur | Points forts |
|---|---|---|
| `text-embedding-3-small` | OpenAI | Rapide, peu coûteux |
| `text-embedding-3-large` | OpenAI | Plus précis |
| `embed-multilingual-v3` | Cohere | Excellent multilingue dont français |
| `all-MiniLM-L6-v2` | Sentence Transformers | Gratuit, open-source, rapide, prototypage (anglais) |
| `nomic-embed-text` | Ollama (local) | Local, multilingue, sans clé API ni donnée envoyée à un tiers |
| `BAAI/bge-m3` | Sentence Transformers | Multilingue, haute qualité |

> [!tip] Trois critères pour choisir
> **La langue** d'abord — un modèle entraîné surtout en anglais traite mal le français, un corpus francophone exige un modèle multilingue explicite, non négociable. **La taille** ensuite — un modèle plus gros capte des nuances plus fines mais produit des vecteurs plus longs, donc un index plus lourd. **L'hébergement** enfin — un modèle servi par Ollama (voir [[Ollama — Index des fiches]]) tourne en local, sans dépendance à une API externe.

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
> Chroma pour apprendre et prototyper. Pinecone ou Qdrant pour un projet en production. Voir [[Qdrant — Index des fiches]] pour un module dédié (installation, collections, filtrage par payload, multi-tenant, production).

## Le chunking — découper les documents

Avant d'indexer, il faut découper les documents en morceaux (**chunks**). La taille impacte directement la qualité.

| Taille chunk | Avantages | Inconvénients |
|---|---|---|
| Petit (128-256 tokens) | Précis, peu de bruit | Peut perdre le contexte |
| Moyen (512-1024 tokens) | Bon équilibre | Standard recommandé |
| Grand (2048+ tokens) | Contexte riche | Moins précis, plus de bruit |

> [!warning] Le chunk size est crucial
> Trop petit = la réponse manque de contexte. Trop grand = des informations non pertinentes polluent la réponse. Le bon réglage dépend de tes documents — teste les deux extrêmes.

## Quatre stratégies de découpage

La taille n'est qu'un paramètre — la **méthode** de découpage compte tout autant :

| Stratégie | Principe | Quand l'utiliser |
|-----------|----------|----------------------|
| **Taille fixe** | Découpe tous les N tokens, sans égard au contenu | Point de départ simple, documents peu structurés |
| **Récursif** | Découpe en cascade sur des séparateurs (paragraphe → phrase → mot), en respectant la structure autant que possible | Amélioration simple et peu coûteuse du découpage fixe — le défaut recommandé dans la plupart des frameworks (LangChain, LlamaIndex) |
| **Structurel** | Suit la structure explicite du document (titres Markdown, sections HTML, articles de loi) | Documentation technique, contenu déjà bien structuré en sections |
| **Sémantique** | Regroupe les phrases par proximité de sens (embeddings), coupe où le sujet change réellement | Documents au contenu dense sans structure exploitable, au prix d'un coût de calcul plus élevé (un embedding par phrase avant même l'indexation finale) |

> [!tip] Le récursif est le compromis par défaut
> Sans contrainte particulière, un découpage récursif (respectant paragraphes puis phrases) donne un meilleur résultat qu'un découpage à taille fixe pur, pour un coût de mise en œuvre quasi identique — c'est le choix par défaut de la plupart des bibliothèques de chunking.

> [!info] Implémentations complètes
> Le code des stratégies taille fixe (avec recouvrement), par phrases et par sections — ainsi que le tableau de décision par type de corpus et le dépannage associé — est détaillé dans [[RAG 09 — Chunking en pratique (taille fixe, phrases, sections)]].
