#ia #agents #autogen #microsoft #multi-agents #pratique

## AutoGen

Framework multi-agents de Microsoft basé sur des **conversations entre agents**. Particulièrement puissant pour la génération et le débogage de code.

## Le concept fondamental

Dans AutoGen, les agents **se parlent entre eux** comme dans une vraie conversation. L'intelligence émerge du dialogue.

```
[UserProxy Agent]          [Assistant Agent]
"Écris une fonction       →
qui trie une liste"
                          ← "Voici le code :
                             def trier(liste): ..."
"Exécute le code et       →
vérifie qu'il fonctionne"
                          ← "Erreur sur les listes vides.
                             Voici la correction..."
"Parfait. Ajoute des      →
tests unitaires"
                          ← "Voici les tests..."
[TERMINATE]
```

## Les types d'agents AutoGen

### ConversableAgent
La brique de base. Tout agent AutoGen en hérite.

```python
from autogen import ConversableAgent

agent = ConversableAgent(
    name="assistant",
    system_message="Tu es un expert en Python.",
    llm_config={"model": "gpt-4o", "api_key": "..."},
    human_input_mode="NEVER"  # ou "ALWAYS" ou "TERMINATE"
)
```

### AssistantAgent
Agent spécialisé pour générer du code et des solutions.

```python
from autogen import AssistantAgent

développeur = AssistantAgent(
    name="développeur",
    system_message="""Tu es un développeur Python senior.
                      Tu écris du code propre et documenté.
                      Tu corriges les erreurs quand on te les signale.""",
    llm_config=llm_config
)
```

### UserProxyAgent
Représente l'utilisateur. Peut **exécuter du code** automatiquement.

```python
from autogen import UserProxyAgent

exécuteur = UserProxyAgent(
    name="exécuteur",
    human_input_mode="TERMINATE",  # demande à l'humain seulement pour terminer
    max_consecutive_auto_reply=10,
    is_termination_msg=lambda x: "TERMINATE" in x.get("content", ""),
    code_execution_config={
        "work_dir": "code_workspace",
        "use_docker": False
    }
)
```

## Exemple complet — duo codeur/testeur

```python
from autogen import AssistantAgent, UserProxyAgent

llm_config = {"model": "gpt-4o", "api_key": "..."}

# Agent qui code
codeur = AssistantAgent(
    name="codeur",
    llm_config=llm_config,
    system_message="Tu es un développeur Python expert. Tu écris du code correct et testé."
)

# Agent qui exécute et valide
testeur = UserProxyAgent(
    name="testeur",
    human_input_mode="NEVER",
    max_consecutive_auto_reply=5,
    code_execution_config={"work_dir": "workspace", "use_docker": False},
    is_termination_msg=lambda x: "TERMINATE" in x.get("content", "")
)

# Lancer la conversation
testeur.initiate_chat(
    codeur,
    message="""Crée une fonction Python qui :
               1. Lit un fichier CSV
               2. Calcule la moyenne de la colonne 'ventes'
               3. Retourne les 5 lignes avec les plus fortes ventes
               Teste-la avec des données fictives."""
)
```

## GroupChat — plusieurs agents en réunion

```python
from autogen import GroupChat, GroupChatManager

# Plusieurs agents
architecte = AssistantAgent("architecte", ...)
développeur = AssistantAgent("développeur", ...)
testeur = AssistantAgent("testeur", ...)
proxy = UserProxyAgent("utilisateur", ...)

# Créer le groupe
groupe = GroupChat(
    agents=[proxy, architecte, développeur, testeur],
    messages=[],
    max_round=20
)

# Manager qui choisit qui parle
manager = GroupChatManager(groupchat=groupe, llm_config=llm_config)

# Lancer
proxy.initiate_chat(manager, message="Construis une API REST pour gérer des todos")
```

## Comparaison AutoGen vs CrewAI

| | AutoGen | CrewAI |
|---|---|---|
| **Paradigme** | Conversations entre agents | Équipe avec rôles |
| **Point fort** | Code, debug, itération | Workflows créatifs, contenu |
| **Prise en main** | Modérée | Facile |
| **Exécution de code** | Native et puissante | Limitée |
| **Flexibilité** | Très haute | Moyenne |

## Cas d'usage typiques

- ✅ Génération et débogage de code automatique
- ✅ Analyse de données avec exécution Python
- ✅ Revue de code par paires d'agents
- ✅ Résolution de problèmes nécessitant plusieurs itérations
- ❌ Workflows créatifs sans code (CrewAI plus adapté)

> [!tip] Le vrai pouvoir d'AutoGen
> C'est l'exécution de code automatique. L'agent écrit du Python, l'exécute, voit l'erreur, corrige, re-exécute — tout seul. C'est comme avoir un développeur junior autonome pour les tâches répétitives.

> [!warning] Surveiller les boucles
> Sans `max_consecutive_auto_reply` et `is_termination_msg` bien configurés, les agents peuvent boucler indéfiniment et consommer beaucoup de tokens.
