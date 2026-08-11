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

## Agent vs Chatbot vs RAG vs Workflow

Ces quatre objets se confondent souvent dans le discours courant, alors qu'ils se distinguent sur une seule question : **qui décide de la prochaine action ?**

| | Chatbot simple | RAG | Workflow | Agent |
|---|-----------------|-----|----------|-------|
| Objectif | Répond à une question | Répond en s'appuyant sur des documents | Enchaîne des étapes prédéfinies | Accomplit une tâche |
| Qui décide de la prochaine étape | — (un seul tour) | Le code (séquence fixe : chercher puis répondre) | Le développeur (graphe écrit à l'avance) | Le LLM, à chaque tour |
| Nombre d'interactions LLM | Une seule | Une seule (après une recherche fixe) | Plusieurs, dans un ordre figé | Plusieurs itérations (boucle) |
| Action sur le monde réel | Aucune | Recherche documentaire seulement | Selon les étapes codées | Appelle des outils, choisis dynamiquement |
| Comportement | Déterministe | Déterministe | Déterministe (chemin figé) | Autonome (et donc risqué) |

> [!info] Un RAG et un workflow ne sont pas des agents
> Dans un RAG (voir [[RAG 01 — Qu'est-ce que le RAG]]), la séquence chercher-puis-répondre est imposée par le code, le modèle ne choisit pas de chercher. Dans un workflow, plusieurs appels LLM s'enchaînent selon un graphe écrit à l'avance par le développeur — le chemin peut être riche, mais il reste figé. Seul l'agent laisse le LLM décider, à chaque tour, de la prochaine action : c'est cette autonomie de décision qui le définit, pas la présence d'une boucle ou de plusieurs appels LLM.

> [!tip] Un agent n'est pas "mieux" qu'un workflow
> L'autonomie a un coût : imprévisibilité, dérives possibles, dépenses en tokens moins prévisibles. Quand le chemin à suivre est connu à l'avance, un workflow est plus simple, plus rapide et plus sûr qu'un agent. Choisir l'agent seulement quand le chemin ne peut pas être écrit à l'avance.

> [!warning] Un chatbot qui "semble" agir n'est pas forcément un agent
> Un chatbot qui affiche un lien ou suggère une action n'est pas un agent tant que c'est l'utilisateur, pas le LLM, qui décide et déclenche cette action. Ce qui définit un agent est la boucle observe → décide → agit → recommence, pas la présence de boutons dans l'interface.

> [!warning] Autonomie ≠ fiabilité
> Plus un agent est autonome, plus il peut faire des erreurs inattendues. Toujours définir des guardrails (limites d'actions, validation humaine pour les actions critiques).

## La boucle propage les erreurs, elle ne les corrige pas

Chaque tour d'un agent part du résultat du tour précédent : une valeur intermédiaire fausse contamine toute la suite du raisonnement, sans que rien ne l'arrête.

```
Tour 1 : Thought "je dois calculer 23 × 17"
         Action   multiplication(23, 17) → Observation 391
Tour 2 : Thought "j'ajoute maintenant 50 à ce résultat"
         Action   addition(299, 50)   ← le modèle a mal recopié 391 en 299
         Observation 349              ← erreur silencieusement propagée
```

> [!warning] Plus la chaîne est longue, plus le risque est grand
> La boucle ne vérifie pas la cohérence entre les tours : rien n'empêche le modèle de mal recopier un résultat intermédiaire au tour suivant. Ce n'est pas un bug d'implémentation, c'est une propriété structurelle de la boucle agent — d'où l'intérêt de modèles capables, d'une validation des résultats d'outils, et d'un périmètre d'action restreint (voir [[Agents 03 — Les outils (Tools)]]).
