#ia #rag #bases #définition

## Qu'est-ce que le RAG ?

RAG = **Retrieval-Augmented Generation**
Retrieval = récupération · Augmented = enrichi · Generation = génération de réponse

Un LLM seul a deux limites fondamentales :
- Son savoir est figé à sa date d'entraînement
- Il ne connaît pas tes données (documents, base clients, wiki interne...)

Le RAG résout ces deux problèmes en donnant au LLM accès à une **mémoire externe consultable en temps réel**.

## Le principe en une phrase

> Au lieu de tout mettre dans le contexte, on cherche d'abord les passages pertinents, puis on les donne au LLM pour qu'il réponde.

## Comment ça marche — les 2 phases

### Phase 1 — Indexation (une seule fois)

```
Tes documents
    ↓
Découpage en morceaux (chunks)
    ↓
Transformation en vecteurs (embeddings)
    ↓
Stockage dans une vector database
```

### Phase 2 — Requête (à chaque question)

```
Ta question
    ↓
Transformation en vecteur
    ↓
Comparaison avec les vecteurs stockés
    ↓
Récupération des passages les plus proches (Retrieval)
    ↓
Injection dans le prompt avec ta question
    ↓
Le LLM répond en se basant sur ces passages (Generation)
```

## Exemple concret

Tu as 500 pages de documentation produit. Tu demandes :
*"Quelle est la procédure de retour pour un article défectueux ?"*

| Sans RAG | Avec RAG |
|---|---|
| Le LLM invente ou dit qu'il ne sait pas | Retrouve les 3 paragraphes pertinents et répond précisément |

## Cas d'usage typiques

- Chatbot sur ta documentation interne
- Assistant qui répond depuis tes emails / CRM
- Moteur de recherche intelligent sur une base de connaissance
- Support client automatisé basé sur tes FAQs
- Assistant juridique sur tes contrats

## La famille RAG

```
Naive RAG          → pipeline basique, point de départ
Advanced RAG       → pre-retrieval + re-ranking
Modular RAG        → modules interchangeables
Agentic RAG        → agent autonome multi-étapes
Graph RAG          → graphe de connaissances et relations
Hybrid RAG         → vectoriel + mots-clés combinés
```

> [!tip] Pour débuter sans coder
> NotebookLM (Google) est un RAG clé en main sur tes propres documents. Gratuit, zéro configuration.

> [!info] Prérequis techniques
> Pour construire un RAG : Python + LlamaIndex ou LangChain + une vector database (Chroma pour débuter).
