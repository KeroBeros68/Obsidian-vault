#ia #langchain #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier les variables du prompt

```python
# ❌ Variable {domaine} dans le template mais pas dans invoke()
prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es expert en {domaine}."),
    ("human", "{question}")
])
chain = prompt | llm | StrOutputParser()

chain.invoke({"question": "Bonjour ?"})
# → KeyError: 'domaine'

# ✅ Toujours fournir toutes les variables OU utiliser .partial()
chain.invoke({"domaine": "Python", "question": "Bonjour ?"})

# Ou fixer domaine à l'avance
chain_python = (prompt.partial(domaine="Python") | llm | StrOutputParser())
chain_python.invoke({"question": "Bonjour ?"})
```

---

## 🪤 Piège 2 — JsonOutputParser qui échoue sur du markdown

```python
# ❌ Le LLM retourne souvent :
# ```json
# {"nom": "Alice"}
# ```
# → JsonOutputParser plante car les backticks ne sont pas du JSON valide

prompt = ChatPromptTemplate.from_messages([
    ("system", "Réponds en JSON."),   # ← trop vague
    ("human", "{texte}")
])
chain = prompt | llm | JsonOutputParser()
# → json.JSONDecodeError fréquent

# ✅ Option 1 : être très explicite dans le prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", """Réponds UNIQUEMENT en JSON valide.
                  AUCUN markdown, AUCUNE explication, AUCUN backtick.
                  Juste l'objet JSON brut."""),
    ("human", "{texte}")
])

# ✅ Option 2 : utiliser with_structured_output() (plus fiable)
llm_structuré = llm.with_structured_output(MonSchémaPydantic)
```

---

## 🪤 Piège 3 — Mémoire in-memory perdue au redémarrage

```python
# ❌ Utiliser InMemoryChatMessageHistory en production
# → Toutes les conversations perdues à chaque restart du serveur

sessions = {}
def obtenir_historique(session_id):
    if session_id not in sessions:
        sessions[session_id] = InMemoryChatMessageHistory()
    return sessions[session_id]
# Restart → sessions = {} → tout perdu ❌

# ✅ Utiliser SQLite ou Redis en production
from langchain_community.chat_message_histories import SQLChatMessageHistory

def obtenir_historique(session_id):
    return SQLChatMessageHistory(
        session_id=session_id,
        connection_string="sqlite:///conversations.db"
    )
```

---

## 🪤 Piège 4 — Docstring d'outil trop vague

```python
# ❌ Le LLM ne sait pas quand utiliser cet outil
@tool
def search(query: str) -> str:
    """Fait une recherche."""  # ← trop vague
    ...

# ✅ Préciser quand utiliser ET quand ne pas utiliser
@tool
def recherche_documents(query: str) -> str:
    """Cherche dans la base documentaire interne de l'entreprise.
    Utilise pour : politiques RH, procédures internes, fiches produit.
    Ne pas utiliser pour : actualités récentes, prix temps réel (utilise recherche_web).
    """
    ...
```

---

## 🪤 Piège 5 — Agent sans limite de récursion

```python
# ❌ Boucle infinie possible → coût illimité en tokens
agent = create_react_agent(llm, tools)
résultat = agent.invoke({"messages": [...]})
# Si l'agent boucle → des milliers d'appels LLM

# ✅ Toujours définir recursion_limit
résultat = agent.invoke(
    {"messages": [...]},
    config={"recursion_limit": 10}   # max 10 itérations
)
```

---

## 🪤 Piège 6 — RunnableParallel implicite mal compris

```python
# Ce code est souvent source de confusion
chain = (
    {
        "contexte": retriever | formater_docs,
        "question": RunnablePassthrough()
    }
    | prompt | llm | StrOutputParser()
)

# ❌ Confusion fréquente : penser que c'est un dict Python normal
# ✅ C'est un RunnableParallel implicite !
# Les deux branches s'exécutent EN PARALLÈLE
# "contexte" reçoit l'input → le retriever le traite
# "question" reçoit l'input → le passe tel quel
# Les deux résultats sont combinés en un dict avant d'aller au prompt

# Pour déboguer, inspecter ce que reçoit le prompt :
test_chain = {
    "contexte": retriever | formater_docs,
    "question": RunnablePassthrough()
}
print(test_chain.invoke("Ma question"))
# → {"contexte": "...", "question": "Ma question"}
```

---

## 🪤 Piège 7 — chunk_size trop grand ou trop petit

```python
# ❌ Trop petit → perd le contexte, réponses incomplètes
splitter = RecursiveCharacterTextSplitter(chunk_size=64, chunk_overlap=0)

# ❌ Trop grand → trop de bruit, réponse noyée dans l'irrelevant
splitter = RecursiveCharacterTextSplitter(chunk_size=4096, chunk_overlap=0)

# ✅ Point de départ recommandé
splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,    # ajuster selon les résultats
    chunk_overlap=50   # toujours ajouter de l'overlap
)
# Tester : poser une question et inspecter les chunks récupérés
# Si la réponse est coupée → augmenter chunk_size
# Si trop de bruit → diminuer chunk_size
```

---

## 🪤 Piège 8 — Ne pas tester le retriever séparément

```python
# ❌ Tester uniquement la réponse finale
réponse = chain_rag.invoke("Ma question")
print(réponse)   # Mauvaise réponse → mais pourquoi ?

# ✅ Tester chaque étape séparément
# Étape 1 : tester le retriever seul
chunks = retriever.invoke("Ma question")
print(f"{len(chunks)} chunks récupérés")
for chunk in chunks:
    print(f"---\n{chunk.page_content[:200]}")

# Si les chunks sont mauvais → problème de retrieval (embeddings, chunk_size...)
# Si les chunks sont bons mais la réponse est mauvaise → problème de prompt ou LLM
```

---

## 🪤 Piège 9 — Streaming sans flush

```python
# ❌ Sans flush → Python bufferise et affiche tout à la fin
for chunk in chain.stream({"question": "..."}):
    print(chunk, end="")   # ← pas de flush → effet non-streaming

# ✅ Toujours flush=True pour le vrai streaming
for chunk in chain.stream({"question": "..."}):
    print(chunk, end="", flush=True)   # ← affichage immédiat token par token
```

---

## 🪤 Piège 10 — Compiler un graphe LangGraph sans checkpointer avant d'utiliser interrupt()

```python
# ❌ Aucun checkpointer : interrupt() n'a rien pour sauvegarder l'état
app = graphe.compile()
app.invoke({"ticket": "..."}, config)  # échoue dès que le nœud atteint interrupt()

# ✅ Le checkpointer est la condition pour suspendre puis reprendre
from langgraph.checkpoint.memory import InMemorySaver
app = graphe.compile(checkpointer=InMemorySaver())
```

> [!warning] Le checkpointer n'est pas optionnel dès qu'un nœud appelle interrupt()
> C'est lui qui sauvegarde l'état du graphe au moment de la pause et permet à un second `invoke()` avec `Command(resume=...)` de reprendre exactement là où l'exécution s'est arrêtée. Voir [[LC 12 — LangGraph — agents avec état]].

---

## 🪤 Piège 11 — La fonction de routage ne renvoie pas le nom exact du nœud

```python
# ❌ "Controle_Humain" ≠ le nom "controle_humain" passé à add_node
def router(state) -> str:
    return "Controle_Humain" if state["action"] in ACTIONS_DESTRUCTIVES else "appliquer"

# ✅ La chaîne renvoyée doit correspondre exactement
def router(state) -> str:
    return "controle_humain" if state["action"] in ACTIONS_DESTRUCTIVES else "appliquer"
```

> [!warning] add_conditional_edges compare des chaînes, pas des intentions
> Si la fonction de routage renvoie une chaîne qui ne correspond à aucun nom passé à `add_node`, le graphe n'atteint jamais ce nœud — sans erreur explicite au moment du routage lui-même. Vérifier que la fonction ne fait que décider (voir [[LC 12 — LangGraph — agents avec état]]) simplifie ce contrôle : une fonction courte et pure est plus facile à relire que la valeur exacte attendue.

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Variable manquante dans invoke() | Fournir toutes les vars ou `.partial()` |
| JsonOutputParser + markdown LLM | Instructions explicites ou `with_structured_output()` |
| Mémoire perdue au restart | SQLite ou Redis en production |
| Docstring d'outil vague | Préciser quand utiliser ET ne pas utiliser |
| Agent sans limite | `config={"recursion_limit": 10}` |
| RunnableParallel mal compris | Tester le dict intermédiaire séparément |
| Mauvais chunk_size | Commencer à 512/50, tester le retriever seul |
| Pas de test par étape | Tester retriever → prompt → LLM séparément |
| Streaming sans flush | Toujours `print(chunk, end="", flush=True)` |
| interrupt() sans checkpointer | Toujours `compile(checkpointer=...)` |
| Routage LangGraph vers un nœud inexistant | La fonction doit renvoyer le nom exact passé à `add_node` |
