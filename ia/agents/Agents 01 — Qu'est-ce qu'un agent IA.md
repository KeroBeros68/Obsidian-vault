#ia #agents #bases #définition

## Qu'est-ce qu'un agent IA ?

Un agent IA est un LLM qui peut **prendre des décisions et agir** dans le monde réel, pas seulement répondre à une question.

## La différence fondamentale

```
LLM classique :
  Entrée → [LLM] → Sortie
  (une seule passe, passif)

Agent IA :
  Objectif → [LLM réfléchit] → Décide d'une action
                  ↑                    ↓
                  └──── Observe le résultat ←── Exécute l'action
  (boucle continue, actif)
```

> La clé : l'agent **observe, décide, agit, recommence** jusqu'à atteindre son objectif.

## Les 4 capacités d'un agent

### 1. Planifier
Décomposer un objectif complexe en étapes.

```
Objectif : "Prépare un rapport sur nos ventes du trimestre"
Plan :
  1. Récupérer les données de ventes
  2. Calculer les métriques clés
  3. Comparer avec le trimestre précédent
  4. Rédiger le rapport
  5. Envoyer par email à l'équipe
```

### 2. Utiliser des outils
Interagir avec le monde extérieur.

```
Outils possibles :
  - Recherche web
  - Lecture / écriture de fichiers
  - Exécution de code Python
  - Appels API (CRM, email, calendrier...)
  - Requêtes base de données
  - Recherche dans un RAG
```

### 3. Mémoriser
Garder le contexte entre les étapes.

```
Court terme  : la conversation en cours
Long terme   : profil utilisateur, préférences
Épisodique   : résultats des étapes précédentes
```

### 4. S'auto-corriger
Évaluer son propre travail et recommencer si nécessaire.

```
Résultat de l'étape 2 → Agent évalue : "Est-ce suffisant ?"
  → Non → Affiner la recherche ou changer d'approche
  → Oui  → Passer à l'étape 3
```

## Analogie concrète

```
LLM seul  = un expert consultant qui répond à tes questions
Agent IA  = un collaborateur autonome à qui tu donnes un objectif
            et qui le mène à bien de A à Z sans te demander
            à chaque étape quoi faire
```

## Exemples d'agents réels

| Agent | Ce qu'il fait |
|---|---|
| Agent de recherche | Cherche sur le web, compile, synthétise une réponse complète |
| Agent de code | Écrit du code, l'exécute, corrige les bugs, itère |
| Agent commercial | Lit les emails clients, met à jour le CRM, envoie des réponses |
| Agent d'analyse | Lit un CSV, génère des graphiques, rédige un rapport PDF |
| Agent DevOps | Surveille les logs, détecte une anomalie, crée un ticket, alerte |

## La boucle ReAct — le schéma de base

ReAct = **Re**asoning + **Act**ing. Le pattern le plus courant pour les agents.

```
[Observation] → Réfléchit (Thought) → Décide une action (Action)
      ↑                                         ↓
      └──────────────── Résultat (Observation) ←┘
```

Exemple :

```
Thought   : "Je dois trouver le prix actuel du pétrole"
Action    : recherche_web("prix pétrole brent aujourd'hui")
Observation: "Le Brent est à 82,4 $/baril ce matin"
Thought   : "J'ai l'info, je peux répondre"
Réponse   : "Le prix du Brent est actuellement de 82,4 $/baril."
```

> [!tip] Pourquoi "agent" et pas "LLM avec outils" ?
> La différence est l'autonomie. Un LLM avec outils attend qu'on lui dise quel outil utiliser. Un agent décide lui-même quels outils utiliser, dans quel ordre, et combien de fois.

> [!warning] Autonomie ≠ fiabilité
> Plus un agent est autonome, plus il peut faire des erreurs inattendues. Toujours définir des guardrails (limites d'actions, validation humaine pour les actions critiques).
