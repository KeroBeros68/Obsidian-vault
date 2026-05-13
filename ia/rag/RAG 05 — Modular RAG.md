#ia #rag #modular #avancé

## Modular RAG

Au lieu d'un pipeline fixe et séquentiel, chaque composant devient un **module interchangeable**. L'architecture devient flexible et adaptable selon les besoins.

## Principe

```
Pipeline fixe (Naive/Advanced RAG) :
Question → Recherche → Re-rank → LLM → Réponse
(toujours le même chemin)

Pipeline modulaire :
Question → [Routeur] → choisit les modules → [Fusionneur] → LLM → Réponse
            ↑                ↑
         décide           module web
         selon la          module DB
         question          module docs
                           module API
```

## Les modules principaux

### Routeur
Analyse la question et décide quels modules utiliser.

```
"Quelle est la météo demain ?"     → module recherche web
"Quel est mon solde ?"             → module base de données
"Résume ce document PDF"           → module documents locaux
"Compare nos prix avec la concurrence" → module web + module DB
```

### Modules de recherche

| Module | Source de données |
|---|---|
| Vectoriel | Vector database (documents indexés) |
| Web search | Internet en temps réel |
| SQL | Base de données relationnelle |
| API | Services externes (CRM, ERP...) |
| Graph | Base de graphe (relations entre entités) |

### Fusionneur (Fusion / Aggregator)
Combine les résultats de plusieurs modules en une réponse cohérente.

```
Résultats module web   +   Résultats module DB
            ↓
      [Fusionneur]
            ↓
  Contexte unifié pour le LLM
```

### Mémoire (optionnel)
Stocke le contexte des échanges précédents pour personnaliser les réponses.

```
Court terme  : les derniers messages de la conversation
Long terme   : profil utilisateur, préférences, historique
Sémantique   : résumés des conversations passées
```

## Exemple d'architecture complète

```
Question utilisateur
        ↓
    [Routeur]
    /    |    \
[Web] [Docs] [DB]
    \    |    /
   [Fusionneur]
        ↓
    [Re-ranker]
        ↓
   [Compresseur]
        ↓
      [LLM]
        ↓
     Réponse
```

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Très flexible, adapté à des cas complexes | Architecture plus difficile à concevoir |
| Peut interroger plusieurs sources | Plus de composants = plus de points de défaillance |
| Facilement extensible | Débogage plus complexe |
| Standard pour les applications IA en prod | Nécessite une bonne orchestration |

## Outils pour implémenter

- **LangChain** : le plus utilisé, très complet pour les pipelines modulaires
- **LlamaIndex** : excellent pour les pipelines centrés sur les données
- **Haystack** (Deepset) : orienté production, bonne gestion des modules

> [!info] LangChain vs LlamaIndex
> LangChain = couteau suisse, bon pour tout. LlamaIndex = spécialiste des données et du RAG. Pour un RAG complexe, LlamaIndex est souvent plus simple.

> [!warning] Complexité vs bénéfice
> Le Modular RAG est justifié si tu as réellement plusieurs sources de données hétérogènes. Pour une seule source homogène, le Naive ou l'Advanced RAG suffisent largement.
