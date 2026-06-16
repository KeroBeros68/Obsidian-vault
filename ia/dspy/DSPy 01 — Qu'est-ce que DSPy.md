#ia #dspy #bases #définition

## Qu'est-ce que DSPy ?

DSPy (Declarative Self-improving Python) est un framework open-source créé par Stanford NLP qui change fondamentalement la façon de construire des systèmes IA. Il permet d'itérer rapidement sur du code modulaire et structuré, plutôt que sur des chaînes de caractères fragiles, et offre des algorithmes qui compilent les programmes IA en prompts et poids efficaces pour les LLM.

## Le problème que DSPy résout

```
Approche classique (prompt engineering) :
  → Tu écris manuellement chaque prompt
  → Tu changes le modèle → tous tes prompts cassent
  → Tu changes les données → les exemples ne marchent plus
  → Système fragile, difficile à maintenir
  → "Prompt archaeology" : personne ne sait pourquoi ça marche

DSPy :
  → Tu déclares QUOI faire (signature), pas COMMENT le prompter
  → DSPy génère et optimise les prompts automatiquement
  → Changer de modèle = 1 ligne de code
  → Les optimiseurs améliorent les performances avec tes données
  → Système modulaire, testable, portable
```

## L'analogie clé

DSPy est au prompt engineering ce que le langage C est à l'assembleur, ou ce que SQL est à l'arithmétique de pointeurs — une abstraction de plus haut niveau.

```
Assembleur → C → Python
Prompt manuel → LangChain (prompts structurés) → DSPy (déclaratif)
```

## Les 3 briques fondamentales

```
┌─────────────────────────────────────────────────────┐
│                      DSPy                            │
│                                                      │
│  SIGNATURE         MODULE            OPTIMIZER       │
│  "quoi faire"      "comment faire"   "améliorer"    │
│                                                      │
│  question → réponse   Predict        BootstrapFewShot│
│  texte → résumé       ChainOfThought MIPROv2         │
│  contexte → label     ReAct          GEPA            │
│                       RAG            BootstrapFT     │
└─────────────────────────────────────────────────────┘
```

### Signature
Déclaration de ce que le module doit faire — inputs et outputs.
```python
"question -> réponse"
"texte, question -> réponse, sources"
```

### Module
Stratégie d'exécution — comment interroger le LLM.
```python
dspy.Predict(signature)        # appel direct
dspy.ChainOfThought(signature) # raisonnement étape par étape
dspy.ReAct(signature, tools)   # agent avec outils
```

### Optimizer
Améliore automatiquement les prompts et/ou les poids en utilisant tes données et ta métrique.
```python
dspy.BootstrapFewShot(metric=ma_métrique)
dspy.MIPROv2(metric=ma_métrique)
```

## DSPy vs LangChain — quand choisir quoi

| Critère | LangChain | DSPy |
|---|---|---|
| **Paradigme** | Prompt engineering explicite | Déclaratif + optimisation automatique |
| **Prompts** | Tu les écris | DSPy les génère et optimise |
| **Dataset requis** | Non | Oui (pour les optimiseurs) |
| **Portabilité modèle** | Moyenne | Excellente |
| **Courbe d'apprentissage** | Moyenne | Plus raide |
| **Meilleur pour** | Prototypage rapide, contrôle fin | Systèmes robustes, optimisation systématique |

> [!tip] DSPy ne remplace pas LangChain
> Ils sont complémentaires. LangChain gère l'orchestration, les outils, la mémoire. DSPy optimise les prompts des modules LLM. On peut les utiliser ensemble.

> [!info] DSPy v2.5 en 2025
> La version 2.5 introduit l'optimisation jointe sur plusieurs modules, de nouveaux outils d'observabilité, et une intégration native avec GPT-4, Claude Sonnet 4, Gemini 2.0 et les modèles locaux via Ollama et vLLM.
