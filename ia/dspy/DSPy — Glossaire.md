#ia #dspy #glossaire #référence

| Terme | Définition |
|---|---|
| **DSPy** | Declarative Self-improving Python. Framework Stanford NLP pour programmer les LLM de façon déclarative et les optimiser automatiquement. |
| **Signature** | Déclaration des inputs et outputs d'un module DSPy. Remplace le prompt manuel. Peut être inline (`"question -> réponse"`) ou classe (`class MaSig(dspy.Signature)`). |
| **InputField** | Champ d'entrée d'une signature DSPy. `dspy.InputField(desc="description")`. |
| **OutputField** | Champ de sortie d'une signature DSPy. `dspy.OutputField(desc="description")`. |
| **Module** | Brique d'exécution DSPy qui implémente une signature. `Predict`, `ChainOfThought`, `ReAct` sont des modules built-in. |
| **Predict** | Module de base DSPy. Appel direct au LLM selon la signature. |
| **ChainOfThought** | Module DSPy qui ajoute un champ `reasoning` caché pour forcer le raisonnement avant la réponse. |
| **ProgramOfThought** | Module DSPy qui génère et exécute du code Python pour résoudre un problème. |
| **ReAct** | Module DSPy implémentant le pattern Reasoning + Acting avec des outils. |
| **MultiChainComparison** | Module DSPy qui génère plusieurs raisonnements (ChainOfThought) et sélectionne le meilleur. |
| **dspy.Module** | Classe de base pour créer des programmes custom. Hériter et implémenter `forward()`. |
| **forward()** | Méthode principale d'un module custom DSPy. Reçoit les inputs, orchestre les sous-modules, retourne une `Prediction`. |
| **Prediction** | Objet retourné par les modules DSPy. Contient tous les champs de sortie accessibles par attribut. |
| **dspy.Example** | Objet représentant un exemple d'entraînement ou d'évaluation. `.with_inputs()` définit les champs d'entrée. |
| **Optimizer** | Algorithme DSPy qui améliore automatiquement un programme selon une métrique. |
| **BootstrapFewShot** | Optimiseur DSPy qui génère des exemples few-shot optimaux depuis des traces réussies. |
| **MIPROv2** | Optimiseur DSPy qui optimise simultanément instructions et exemples few-shot. Standard recommandé en production. |
| **GEPA** | Generative Evolutionary Prompt Architect. Optimiseur DSPy génétique — meilleur résultat, plus coûteux. |
| **BootstrapFinetune** | Optimiseur DSPy qui fine-tune les poids du LLM à partir des traces du programme. |
| **compile()** | Méthode de l'optimiseur pour optimiser un programme DSPy sur un trainset. `optimiseur.compile(programme, trainset=trainset)`. |
| **dspy.Evaluate** | Classe DSPy pour évaluer un programme sur un devset avec une métrique. |
| **Métrique DSPy** | Fonction `(exemple, prédiction, trace=None) -> bool | float` qui évalue la qualité d'une prédiction. |
| **Trace** | Enregistrement d'une exécution réussie d'un programme DSPy (input → output). Utilisé par les optimiseurs pour générer les exemples. |
| **dspy.Assert** | Assertion DSPy stricte qui relance le LLM si la contrainte n'est pas respectée. |
| **dspy.Suggest** | Assertion DSPy souple qui guide le LLM sans bloquer si la suggestion n'est pas suivie. |
| **dspy.inspect_history** | Affiche les prompts et réponses des derniers appels LLM. Essentiel pour déboguer. |
| **dspy.configure** | Configure le LM global et d'autres paramètres de DSPy. |
| **dspy.context** | Context manager pour overrider temporairement le LM ou d'autres paramètres. |
| **auto parameter** | Paramètre de MIPROv2 qui contrôle le budget d'optimisation : `"light"` (~$0.5), `"medium"` (~$2), `"heavy"` (~$10). |
| **Demo** | Exemple few-shot ajouté par un optimiseur DSPy dans le prompt d'un module. |
| **Bootstrapping (DSPy)** | Phase des optimiseurs où DSPy exécute le programme sur différents inputs pour collecter des traces de comportement réussi. |
| **LiteLLM** | Librairie utilisée par DSPy sous le capot pour unifier l'accès à tous les LLM providers. |
| **Portabilité de modèle** | Capacité de DSPy à changer de LLM sans modifier le code du programme — juste `dspy.configure(lm=nouveau_lm)`. |
