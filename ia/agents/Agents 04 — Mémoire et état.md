#ia #agents #mémoire #état #bases

## Mémoire et état d'un agent

Par défaut, un LLM n'a aucune mémoire entre les appels. Un agent a besoin de différents types de mémoire pour être réellement utile sur des tâches longues.

## Les 4 types de mémoire

```
┌─────────────────────────────────────────────────────┐
│                   MÉMOIRE AGENT                      │
│                                                      │
│  ┌────────────────┐    ┌────────────────────────┐   │
│  │  IN-CONTEXT    │    │      EXTERNE           │   │
│  │  (temporaire)  │    │     (persistante)      │   │
│  │                │    │                        │   │
│  │ • Conversation │    │ • Profil utilisateur   │   │
│  │ • Étapes       │    │ • Historique sessions  │   │
│  │   précédentes  │    │ • Base de connaissances│   │
│  └────────────────┘    └────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### 1. Mémoire in-context (conversation)

Tout ce qui est dans la fenêtre de contexte du LLM lors de l'appel courant.

```
[System prompt]
[Objectif initial]
[Étape 1] Action → Résultat
[Étape 2] Action → Résultat
[Étape 3] Action → Résultat  ← on est ici
```

- ✅ Accessible instantanément par le LLM
- ❌ Limitée par la context window (128k, 200k tokens selon le modèle)
- ❌ Perdue à la fin de la session

> [!warning] Attention à la context window
> Sur des agents qui tournent longtemps, l'historique des actions peut rapidement dépasser la context window. Prévoir une stratégie de résumé ou de compression.

### 2. Mémoire épisodique (résumés)

Résumés compressés des échanges passés, stockés en dehors du contexte.

```
Session 1 (complète, 50 000 tokens)
    ↓
Résumé automatique (500 tokens)
    ↓
Stocké → Réinjecté au début de la session 2
```

### 3. Mémoire sémantique (RAG)

Base de connaissances vectorielle que l'agent peut interroger.

```
Agent a besoin d'une information → appelle l'outil recherche_mémoire(query)
    ↓
Recherche dans la vector DB → retourne les passages pertinents
    ↓
Injecté dans le contexte
```

C'est la jonction entre les agents et le RAG.

### 4. Mémoire procédurale (état)

L'état courant de la tâche en cours — où en est l'agent dans son plan.

```python
état = {
  "objectif": "Préparer le rapport Q1",
  "étapes_complétées": ["récupération_données", "calcul_métriques"],
  "étape_courante": "génération_graphiques",
  "étapes_restantes": ["rédaction", "envoi"],
  "données_intermédiaires": {
    "revenus_q1": 1250000,
    "croissance": "+12%"
  }
}
```

## Stratégies de gestion de la mémoire longue

### Résumé progressif
```
Toutes les N étapes → LLM résume l'historique → Remplace les détails par le résumé
```

### Fenêtre glissante
```
Garder seulement les K dernières étapes dans le contexte
+ Résumé des étapes plus anciennes
```

### Extraction sélective
```
À chaque étape, extraire les informations clés dans une structure
et supprimer les détails bruts
```

## Persistance entre sessions

Pour qu'un agent "se souvienne" d'une session à l'autre.

```
Fin de session → Sauvegarder dans une base de données :
  - Résumé de la session
  - Informations importantes sur l'utilisateur
  - Préférences détectées
  - Tâches en cours et leur état

Début de session → Charger et injecter dans le system prompt
```

Outils : Redis, PostgreSQL, SQLite, Pinecone (pour la mémoire sémantique).

## Frameworks et mémoire

| Framework | Gestion mémoire |
|---|---|
| **LangChain** | ConversationBufferMemory, SummaryMemory, VectorStoreMemory |
| **LangGraph** | State graph — état explicite et persistant entre nœuds |
| **Mem0** | Librairie dédiée à la mémoire longue terme pour agents |
| **LlamaIndex** | ChatMemoryBuffer, VectorMemory |

> [!tip] Mem0 — la librairie spécialisée
> Mem0 est une librairie open-source dédiée à la mémoire longue terme des agents. Elle gère automatiquement l'extraction et la récupération des informations importantes.
