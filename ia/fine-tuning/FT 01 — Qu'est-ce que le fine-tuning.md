#ia #fine-tuning #bases #définition

## Qu'est-ce que le fine-tuning ?

Le fine-tuning est le processus d'**entraînement supplémentaire** d'un modèle de fondation existant sur un dataset spécifique, pour le spécialiser dans un domaine ou un style particulier.

## L'analogie du stagiaire expert

```
Modèle de fondation (pré-entraîné) :
  = Un diplômé très cultivé qui sait tout en général
  Parle 50 langues, connaît tous les sujets
  Mais ne connaît pas ta façon de travailler

Fine-tuning :
  = Tu le formes à tes méthodes pendant 3 mois
  Il apprend ton style, ton vocabulaire, tes processus
  Résultat : un expert généraliste + tes pratiques spécifiques
```

## Ce qui change dans le modèle

```
Avant fine-tuning :
  Modèle de fondation
  Poids W₁, W₂, W₃... (milliards de paramètres)
  Comportement généraliste

Pendant fine-tuning :
  On continue l'entraînement sur tes données
  Les poids s'ajustent légèrement : W₁', W₂', W₃'...
  Le gradient descent minimise l'erreur sur tes exemples

Après fine-tuning :
  Modèle spécialisé
  Mêmes poids de base + ajustements spécifiques à ton domaine
  Comportement adapté à ton cas d'usage
```

## Ce que le fine-tuning peut modifier

```
Style et ton        : toujours répondre comme ta marque, formel/informel
Format de sortie    : toujours en JSON, toujours en bullet points
Vocabulaire métier  : maîtriser ton jargon spécifique
Comportement        : toujours poser une question de suivi, toujours citer
Domaine spécifique  : droit, médecine, finance, jeu vidéo...
Langue/dialecte     : argot, langue régionale, style très spécifique
Personnalité        : un assistant avec une vraie identité de marque
```

## Ce que le fine-tuning ne fait PAS

```
❌ Ajouter des connaissances récentes  → utiliser RAG
❌ Accéder à tes données en temps réel → utiliser RAG ou MCP
❌ Mémoriser des faits précis          → les LLM mémorisent mal les facts
❌ Garantir zéro hallucination         → aucune technique ne garantit ça
❌ Remplacer un bon prompt             → souvent le prompt suffit
```

> [!warning] Le fine-tuning ne donne pas de "mémoire"
> Fine-tuner sur "notre produit s'appelle X et coûte Y€" ne garantit pas que le modèle le restituera toujours correctement. Pour des faits précis → RAG.

## Les 3 types de fine-tuning

```
1. Supervised Fine-Tuning (SFT)
   Apprend depuis des paires (entrée, sortie idéale)
   Le plus courant, le plus simple
   "Voici des exemples de ce que tu dois faire"

2. RLHF (Reinforcement Learning from Human Feedback)
   Apprend depuis des préférences humaines (A est meilleur que B)
   Utilisé par OpenAI/Anthropic pour aligner les modèles
   "Voici ce que les humains préfèrent"

3. RLAIF (Reinforcement Learning from AI Feedback)
   Comme RLHF mais le feedback vient d'un autre LLM
   Moins coûteux que RLHF, résultats proches
   "Voici ce qu'un LLM évaluateur préfère"
```

## La hiérarchie des techniques de personnalisation

```
Complexité croissante →

Prompting  →  Few-shot  →  RAG  →  Fine-tuning  →  Entraînement from scratch
   │              │          │          │                    │
Gratuit        Gratuit    Moyen      Élevé              Très élevé
Immédiat       Immédiat  Rapide    Jours/semaines      Mois/années
Aucune        Aucune    Données     Dataset             Dataset
donnée         donnée    indexées   labellisé           massif
```

> [!tip] Règle du 95%
> Dans 95% des cas, un bon prompt + RAG donne des résultats suffisants sans avoir besoin de fine-tuning. Commence toujours par là avant d'investir dans le fine-tuning.
