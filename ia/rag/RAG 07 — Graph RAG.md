#ia #rag #graph #graphe #avancé

## Graph RAG

Au lieu de chercher des passages de texte, le Graph RAG cherche dans un **graphe de connaissances** — un réseau de relations entre entités. Idéal pour des questions qui nécessitent de naviguer entre des entités liées.

## Limite du RAG classique

Le RAG classique traite le texte comme des blocs indépendants. Il perd les **relations** entre les entités.

```
Texte : "Einstein a travaillé avec Bohr à Copenhague. Bohr a enseigné à Heisenberg."

RAG classique stocke 2 chunks séparés.
Une question sur le lien Einstein → Heisenberg raterait la connexion.

Graph RAG voit :
Einstein ──[a travaillé avec]──> Bohr
Bohr     ──[a enseigné]──────> Heisenberg
Bohr     ──[basé à]──────────> Copenhague
→ Peut répondre : "Einstein et Heisenberg sont liés via Bohr"
```

## Structure d'un graphe de connaissances

### Nœuds (entités)
Les "choses" : personnes, entreprises, produits, concepts, lieux, dates...

### Arêtes (relations)
Les liens entre entités : travaille_pour, est_un, a_créé, est_lié_à, date_de...

```
[Einstein] ──[a développé]──> [Relativité restreinte]
[Einstein] ──[a reçu]───────> [Prix Nobel Physique 1921]
[Einstein] ──[né à]──────────> [Ulm, Allemagne]
[Relativité] ──[publiée en]──> [1905]
[Relativité] ──[liée à]──────> [Énergie]
```

## Pipeline Graph RAG

```
[Documents]
    ↓
Extraction d'entités et relations (via LLM)
    ↓
Construction du graphe (Neo4j, NetworkX...)
    ↓ (indexation)

[Question]
    ↓
Identification des entités dans la question
    ↓
Navigation dans le graphe (traversée des relations)
    ↓
Récupération des sous-graphes pertinents
    ↓
Conversion en contexte texte
    ↓
[LLM] → Réponse
```

## Comparaison avec le RAG classique

| | RAG Classique | Graph RAG |
|---|---|---|
| **Stockage** | Chunks de texte | Entités + relations |
| **Recherche** | Similarité vectorielle | Traversée de graphe |
| **Forces** | Questions directes | Questions relationnelles |
| **Exemple fort** | "Qu'est-ce que la relativité ?" | "Qui a influencé Einstein ?" |
| **Complexité** | Moyenne | Élevée |

## Cas d'usage idéaux

- ✅ Bases de connaissances complexes (encyclopédies, wikis techniques)
- ✅ Données avec beaucoup de relations (organigrammes, généalogies, réseaux)
- ✅ Questions du type "quels sont tous les X liés à Y ?"
- ✅ Recommandation (produits liés, articles similaires)
- ❌ Documents simples sans relations entre entités
- ❌ Cas où la structure relationnelle n'apporte pas de valeur

## Outils

| Outil | Usage |
|---|---|
| **Neo4j** | Base de graphe la plus utilisée, avec langage Cypher |
| **Microsoft GraphRAG** | Implémentation open-source de Microsoft |
| **LlamaIndex** | Support natif du Graph RAG |
| **NetworkX** | Librairie Python pour graphes (prototypage) |

> [!info] Microsoft GraphRAG
> Microsoft a publié en 2024 une implémentation open-source de GraphRAG très complète. C'est un bon point de départ pour explorer ce paradigme.

> [!warning] Coût d'extraction
> Construire le graphe nécessite de passer tous les documents dans un LLM pour extraire les entités et relations. Coûteux sur de grands corpus.
