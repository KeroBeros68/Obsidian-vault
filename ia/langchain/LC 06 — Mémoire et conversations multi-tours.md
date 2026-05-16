#ia #langchain #mémoire #conversations #bases

## Mémoire et conversations multi-tours

Par défaut, chaque appel LLM est stateless. La mémoire injecte l'historique dans chaque nouvelle requête pour maintenir le contexte.

## Le problème

```
Sans mémoire :
  Tour 1 → "Je m'appelle Alice"    → "Bonjour Alice !"
  Tour 2 → "Quel est mon prénom ?" → "Je ne sais pas."  ❌

Avec mémoire :
  Tour 1 → "Je m'appelle Alice"    → "Bonjour Alice !"
  Tour 2 → "Quel est mon prénom ?" → "Tu t'appelles Alice." ✅
```

## Brique de base — InMemoryChatMessageHistory

```python
from langchain_core.chat_history import InMemoryChatMessageHistory

historique = InMemoryChatMessageHistory()

# Ajouter des messages
historique.add_user_message("Je m'appelle Alice")
historique.add_ai_message("Bonjour Alice !")

# Lire l'historique
for msg in historique.messages:
    print(f"{msg.type}: {msg.content}")

# Vider
historique.clear()
```

## RunnableWithMessageHistory — connecter à une chain

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.output_parsers import StrOutputParser
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-sonnet-4-20250514")

# 1. Prompt avec placeholder pour l'historique
prompt = ChatPromptTemplate.from_messages([
    ("system", "Tu es un assistant utile et mémorisateur."),
    MessagesPlaceholder(variable_name="history"),  # ← historique injecté ici
    ("human", "{input}")
])

chain = prompt | llm | StrOutputParser()

# 2. Stockage par session
sessions = {}
def obtenir_historique(session_id: str) -> InMemoryChatMessageHistory:
    if session_id not in sessions:
        sessions[session_id] = InMemoryChatMessageHistory()
    return sessions[session_id]

# 3. Envelopper la chain
chain_mémo = RunnableWithMessageHistory(
    chain,
    obtenir_historique,
    input_messages_key="input",
    history_messages_key="history"
)

# 4. Utiliser — session_id isole chaque conversation
config_alice = {"configurable": {"session_id": "alice"}}
config_bob   = {"configurable": {"session_id": "bob"}}

chain_mémo.invoke({"input": "Je m'appelle Alice"}, config=config_alice)
chain_mémo.invoke({"input": "J'ai 30 ans"}, config=config_alice)
r = chain_mémo.invoke({"input": "Quel est mon prénom et mon âge ?"}, config=config_alice)
print(r)  # → "Tu t'appelles Alice et tu as 30 ans." ✅

# Bob a son propre historique séparé
chain_mémo.invoke({"input": "Je suis Bob"}, config=config_bob)
```

## Window Memory — garder les N derniers messages

```python
from langchain_core.chat_history import InMemoryChatMessageHistory

class WindowChatHistory(InMemoryChatMessageHistory):
    """Historique limité aux N derniers messages."""
    def __init__(self, max_messages: int = 10):
        super().__init__()
        self.max_messages = max_messages

    def add_messages(self, messages):
        super().add_messages(messages)
        if len(self.messages) > self.max_messages:
            self.messages = self.messages[-self.max_messages:]

sessions_window = {}
def obtenir_historique_window(session_id: str):
    if session_id not in sessions_window:
        sessions_window[session_id] = WindowChatHistory(max_messages=6)
    return sessions_window[session_id]
```

## Summary Memory — résumer les anciens messages

```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.messages import SystemMessage

class SummaryChatHistory(InMemoryChatMessageHistory):
    """Résume automatiquement les messages anciens."""
    def __init__(self, max_messages: int = 10, llm=None):
        super().__init__()
        self.max_messages = max_messages
        self.llm = llm
        self.résumé = ""

    def add_messages(self, messages):
        super().add_messages(messages)
        if len(self.messages) > self.max_messages:
            anciens = self.messages[:-self.max_messages]
            récents = self.messages[-self.max_messages:]
            texte = "\n".join([f"{m.type}: {m.content}" for m in anciens])
            prompt = f"Résume en 3 phrases : {texte}\nRésumé existant : {self.résumé}"
            self.résumé = self.llm.invoke(prompt).content
            self.messages = [SystemMessage(content=f"Résumé: {self.résumé}")] + récents
```

## Persistance SQLite — survive aux redémarrages

```python
from langchain_community.chat_message_histories import SQLChatMessageHistory

def obtenir_historique_sql(session_id: str):
    return SQLChatMessageHistory(
        session_id=session_id,
        connection_string="sqlite:///conversations.db"
    )

chain_persistant = RunnableWithMessageHistory(
    chain,
    obtenir_historique_sql,
    input_messages_key="input",
    history_messages_key="history"
)

# La conversation survit aux redémarrages du programme
config = {"configurable": {"session_id": "user_42"}}
chain_persistant.invoke({"input": "Je m'appelle Alice"}, config=config)
# Redémarrer le programme...
r = chain_persistant.invoke({"input": "Tu te souviens de mon prénom ?"}, config=config)
# → "Oui, tu t'appelles Alice !" ✅
```

## Persistance Redis — production multi-serveurs

```python
from langchain_community.chat_message_histories import RedisChatMessageHistory

def obtenir_historique_redis(session_id: str):
    return RedisChatMessageHistory(
        session_id=session_id,
        url="redis://localhost:6379",
        ttl=3600   # expiration après 1h d'inactivité
    )
```

## Chatbot complet avec gestion des sessions

```python
class Chatbot:
    def __init__(self, system_prompt: str = "Tu es un assistant utile."):
        self.llm = ChatAnthropic(model="claude-sonnet-4-20250514")
        self.sessions = {}

        prompt = ChatPromptTemplate.from_messages([
            ("system", system_prompt),
            MessagesPlaceholder("history"),
            ("human", "{input}")
        ])

        self.chain = RunnableWithMessageHistory(
            prompt | self.llm | StrOutputParser(),
            self._obtenir_historique,
            input_messages_key="input",
            history_messages_key="history"
        )

    def _obtenir_historique(self, session_id: str):
        if session_id not in self.sessions:
            self.sessions[session_id] = InMemoryChatMessageHistory()
        return self.sessions[session_id]

    def chat(self, message: str, session_id: str = "default") -> str:
        return self.chain.invoke(
            {"input": message},
            config={"configurable": {"session_id": session_id}}
        )

    def reset(self, session_id: str = "default"):
        if session_id in self.sessions:
            self.sessions[session_id].clear()

    def historique(self, session_id: str = "default") -> list:
        return self.sessions.get(session_id, InMemoryChatMessageHistory()).messages


bot = Chatbot("Tu es un assistant Python expert.")
print(bot.chat("Je débute en Python", "alice"))
print(bot.chat("Par quoi commencer ?", "alice"))
print(bot.chat("Résume nos échanges", "alice"))
bot.reset("alice")
```

## Choisir son stockage

| Stockage | Usage |
|---|---|
| `InMemoryChatMessageHistory` | Prototype, tests |
| `WindowChatHistory` | Conversations longues, contexte limité |
| `SummaryChatHistory` | Très longues sessions |
| `SQLChatMessageHistory` | Persistance locale simple |
| `RedisChatMessageHistory` | Production multi-serveurs |

> [!tip] session_id = identifiant utilisateur
> En production, le `session_id` est typiquement l'ID utilisateur de ton système d'auth. Chaque utilisateur a son propre historique isolé.

> [!warning] La mémoire in-memory disparaît au redémarrage
> `InMemoryChatMessageHistory` est parfait pour les tests mais ne convient pas à la production. Utilise SQLite ou Redis dès que tu as besoin de persistance.
