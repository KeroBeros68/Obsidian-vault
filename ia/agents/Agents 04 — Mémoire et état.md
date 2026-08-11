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

> [!info] Mem0 vs Letta : une brique ou un cadre complet
> Mem0 se concentre sur la seule couche mémoire, à brancher sur l'agent de son choix quel que soit son framework — c'est une brique, pas un cadre complet. **Letta** (anciennement MemGPT) va plus loin : un framework d'agents construit *autour* de la mémoire, avec une gestion hiérarchique inspirée d'un système d'exploitation où l'agent décide lui-même ce qu'il garde en contexte et ce qu'il archive — plus intégré, mais plus engageant. Le choix suit le besoin : ajouter de la mémoire à un agent existant appelle une brique comme Mem0 ; concevoir un agent dont la mémoire est le cœur justifie un framework comme Letta (non couvert dans ce vault, voir [[Manques]]).

## Construire une mémoire long terme avec Mem0

Écrire à la main l'extraction des faits, leur vectorisation et leur stockage serait long. Mem0 assemble trois briques dans une configuration : un LLM pour extraire les faits d'une conversation, un embedder pour les vectoriser, et un vector store pour les conserver — ici, tout pointe vers des services locaux (Ollama et [[Qdrant — Index des fiches|Qdrant]]).

```python
CONFIG = {
    "llm": {
        "provider": "ollama",
        "config": {"model": "qwen2.5", "ollama_base_url": "http://localhost:11434"},
    },
    "embedder": {
        "provider": "ollama",
        "config": {
            "model": "nomic-embed-text",
            "ollama_base_url": "http://localhost:11434",
            "embedding_dims": 768,
        },
    },
    "vector_store": {
        "provider": "qdrant",
        "config": {
            "collection_name": "agent_memoire",
            "host": "localhost",
            "port": 6333,
            "embedding_model_dims": 768,
        },
    },
}

from mem0 import Memory

def ouvrir_memoire() -> Memory:
    """Ouvre la mémoire Mem0 : Qdrant la rend persistante entre les runs."""
    return Memory.from_config(CONFIG)
```

> [!warning] La dimension des vecteurs doit être identique partout
> `embedding_dims` (config de l'embedder) et `embedding_model_dims` (config de la collection Qdrant) doivent porter la même valeur — 768 pour `nomic-embed-text`. Un écart entre les deux fait échouer le stockage. C'est un cas particulier du principe déjà vu dans [[RAG — Pièges classiques]] (Piège 8) : les vecteurs d'un même espace doivent tous partager la même origine et la même dimension.

### Mémoriser et rappeler, toujours filtrés par utilisateur

La mémoire s'utilise par deux opérations : `add` range les faits d'un échange, `search` retrouve les souvenirs pertinents pour une question. Les deux sont systématiquement filtrées par `user_id` — chaque utilisateur a sa propre mémoire, jamais mélangée à celle d'un autre (le même principe de cloisonnement que le filtrage par payload vu dans [[Qdrant 04 — Rechercher & filtrer par payload]]).

```python
def memoriser(memoire, user_id, echange):
    """Range les faits marquants d'un échange dans la mémoire long terme."""
    memoire.add(echange, user_id=user_id)

def repondre(memoire, user_id, question):
    """Répond à une question en s'appuyant sur la mémoire long terme."""
    souvenirs = memoire.search(question, user_id=user_id, limit=5)["results"]
    contexte = "\n".join(f"- {s['memory']}" for s in souvenirs)
    # contexte est injecté dans le prompt système du modèle...
```

> [!tip] La mémoire long terme survit au redémarrage du processus
> Lors d'une première session, l'utilisateur dit « je préfère coder en Python » : Mem0 en extrait le fait et le range dans Qdrant. Lors d'une seconde session — une instance `Memory` toute neuve, comme après un redémarrage — la question « quel langage est-ce que je préfère ? » déclenche un `search` qui retrouve le fait dans Qdrant. La mémoire ne vit pas dans le processus Python, mais dans Qdrant : le processus peut redémarrer, l'agent garde son passé.

### Compaction et oubli

Une mémoire qui ne fait que grossir finit par poser problème : trop de souvenirs, des recherches plus lentes, des faits périmés qui polluent les réponses.

La **compaction** fusionne ou résume des souvenirs redondants — dix échanges sur le même sujet peuvent se condenser en un fait synthétique. Mem0 applique déjà une forme de cela : il met à jour un fait existant plutôt que d'en empiler un quasi-identique.

L'**oubli** est l'autre face : retirer délibérément des souvenirs devenus faux (« je travaillais sur Kubernetes » n'est plus vrai) ou sans intérêt. Décider quoi oublier — par ancienneté, par pertinence, sur demande de l'utilisateur — fait partie de la conception d'un agent à mémoire : une mémoire n'est utile que si elle reste juste.

## Dépannage (Mem0 + Qdrant)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Erreur de dimension au stockage | Embedder et collection Qdrant divergent | Aligner `embedding_dims` et `embedding_model_dims` |
| `Connection refused` sur le port 6333 | Qdrant n'est pas démarré | Lancer le conteneur `qdrant/qdrant` |
| Mem0 réclame d'installer `ollama` | Bibliothèque cliente absente | Installer le paquet `ollama` |
| `search` ne retrouve rien | `user_id` différent entre `add` et `search` | Utiliser le même `user_id` partout |
| L'agent mélange deux utilisateurs | Recherche non filtrée | Toujours passer `user_id` à `search` |
