#ia #rag #agentic #agents #avancé

## Agentic RAG

Le RAG où un agent **décide lui-même** combien de fois chercher, quoi chercher, et vérifie ses propres réponses. Le système devient autonome et capable de gérer des questions complexes en plusieurs étapes.

## Différence fondamentale

```
RAG classique :
Question → 1 recherche → Réponse
(pipeline fixe, toujours la même mécanique)

Agentic RAG :
Question → Agent réfléchit → Décide quoi faire
                ↓
         Cherche si nécessaire
                ↓
         Évalue le résultat
                ↓ (si insuffisant)
         Cherche encore avec une meilleure stratégie
                ↓ (si suffisant)
         Génère la réponse finale
```

## Les capacités de l'agent

### Planification
L'agent décompose une question complexe en sous-questions.

```
Question : "Compare nos performances commerciales Q1 vs Q2 et identifie les causes des écarts"

Agent planifie :
  1. Récupérer les données de ventes Q1
  2. Récupérer les données de ventes Q2
  3. Calculer les écarts par produit / région
  4. Chercher dans les rapports les événements Q1 et Q2
  5. Croiser les données pour identifier les causes
  6. Synthétiser
```

### Self-correction (auto-correction)
L'agent évalue sa propre réponse et recommence si elle est insuffisante.

```
Réponse générée → Agent vérifie :
  "Cette réponse est-elle complète ?"
  "Est-elle bien sourcée ?"
  "Y a-t-il des contradictions ?"
        ↓
  Si non satisfaisante → nouvelle recherche ciblée
  Si satisfaisante     → réponse finale
```

### Utilisation d'outils
L'agent peut appeler des outils en dehors de la recherche vectorielle.

```
Outils disponibles :
  - Recherche vectorielle dans les documents
  - Recherche web en temps réel
  - Calculatrice / exécution de code
  - Appels API externes
  - Lecture / écriture de fichiers
```

## Exemple de trace d'exécution

```
Question : "Quel est le ROI de notre campagne email de mars ?"

[Étape 1] Agent → cherche "campagne email mars statistiques"
          → résultats : taux d'ouverture, clics, mais pas de données financières

[Étape 2] Agent → cherche "chiffre d'affaires mars canal email"
          → résultats : revenus attribués au canal email

[Étape 3] Agent → appelle calculatrice
          ROI = (revenus - coûts) / coûts × 100

[Étape 4] Agent → vérifie la cohérence des chiffres
          → cohérent → génère la réponse finale
```

## Frameworks pour Agentic RAG

| Framework | Points forts |
|---|---|
| **LangGraph** (LangChain) | Graphe d'états, très flexible |
| **LlamaIndex Workflows** | Bien intégré avec le RAG |
| **AutoGen** (Microsoft) | Multi-agents qui collaborent |
| **CrewAI** | Agents avec rôles définis (comme une équipe) |

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Gère des questions très complexes | Latence élevée (plusieurs appels LLM) |
| S'adapte à chaque situation | Coût plus important |
| Peut utiliser de nombreux outils | Comportement moins prévisible |
| Auto-correction améliore la fiabilité | Débogage difficile |

> [!warning] Non-déterminisme
> Un agent peut produire des réponses différentes à la même question selon le chemin qu'il emprunte. À prendre en compte dans les cas d'usage critiques.

> [!tip] Cas d'usage idéaux
> Questions multi-étapes, analyse de données complexes, tâches nécessitant plusieurs sources hétérogènes, automatisation de workflows complets.
