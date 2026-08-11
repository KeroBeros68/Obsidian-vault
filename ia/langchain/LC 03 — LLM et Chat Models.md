#ia #langchain #llm #chatmodels #bases

## LLM et Chat Models

LangChain fournit une interface unifiée pour tous les LLM. Changer de modèle = changer une ligne de code.

## Les deux familles

```
LLM           : prend un string → retourne un string (ancienne API)
ChatModel     : prend des messages → retourne un AIMessage (standard actuel)

En pratique : toujours utiliser les ChatModels (Claude, GPT-4, Gemini...)
```

## Les principaux Chat Models

```python
# Claude (Anthropic)
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-sonnet-4-20250514", temperature=0.7)

# GPT (OpenAI)
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4o", temperature=0.7)

# Gemini (Google)
from langchain_google_genai import ChatGoogleGenerativeAI
llm = ChatGoogleGenerativeAI(model="gemini-1.5-pro")

# Mistral
from langchain_mistralai import ChatMistralAI
llm = ChatMistralAI(model="mistral-large-latest")

# Modèle local via vLLM (API compatible OpenAI)
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(
    model="Qwen/Qwen3-8B",
    base_url="http://localhost:8000/v1",
    api_key="no-key"
)

# Modèle local via Ollama
from langchain_community.chat_models import ChatOllama
llm = ChatOllama(model="mistral")
```

> [!info] Installer et configurer Ollama
> Voir [[Ollama — Index des fiches]] pour l'installation, le choix du modèle et la sécurisation de l'API locale avant de la brancher sur LangChain.

## Paramètres clés

```python
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(
    model="claude-sonnet-4-20250514",
    temperature=0.3,      # 0=déterministe, 1=créatif
    max_tokens=1000,      # limite la longueur de la réponse
    timeout=30,           # timeout en secondes
    max_retries=3,        # nombre de retries en cas d'erreur
)
```

## Modes d'invocation

```python
# ── invoke() — appel synchrone standard ──────────────────
réponse = llm.invoke("Bonjour !")
print(réponse.content)   # → "Bonjour ! Comment puis-je vous aider ?"
print(type(réponse))     # → AIMessage

# ── stream() — réponse au fur et à mesure ────────────────
for chunk in llm.stream("Explique le RAG en 3 phrases"):
    print(chunk.content, end="", flush=True)
# → Les mots apparaissent progressivement

# ── ainvoke() — asynchrone ───────────────────────────────
import asyncio
async def appel_async():
    réponse = await llm.ainvoke("Bonjour !")
    return réponse.content

asyncio.run(appel_async())

# ── batch() — plusieurs appels en parallèle ──────────────
questions = [
    "Capitale de la France ?",
    "Capitale de l'Allemagne ?",
    "Capitale de l'Italie ?"
]
réponses = llm.batch(questions)
for r in réponses:
    print(r.content)
```

## Passer des messages directement

```python
from langchain_core.messages import SystemMessage, HumanMessage, AIMessage

messages = [
    SystemMessage(content="Tu es un assistant expert en Python."),
    HumanMessage(content="C'est quoi une list comprehension ?"),
    AIMessage(content="C'est une syntaxe compacte pour créer des listes..."),
    HumanMessage(content="Donne-moi un exemple.")
]

réponse = llm.invoke(messages)
print(réponse.content)
```

## Structured Output — forcer un format de sortie

Méthode moderne pour obtenir une sortie structurée fiable.

```python
from pydantic import BaseModel, Field
from langchain_anthropic import ChatAnthropic

class Analyse(BaseModel):
    sentiment: str = Field(description="positif, négatif ou neutre")
    score: float = Field(description="score de 0 à 1")
    mots_clés: list[str] = Field(description="3 mots clés principaux")
    résumé: str = Field(description="résumé en une phrase")

llm = ChatAnthropic(model="claude-sonnet-4-20250514")

# with_structured_output force le LLM à respecter le schéma Pydantic
llm_structuré = llm.with_structured_output(Analyse)

résultat = llm_structuré.invoke(
    "Analyse ce texte : 'LangChain est un framework incroyable, j'adore l'utiliser !'"
)

print(type(résultat))         # → <class 'Analyse'>
print(résultat.sentiment)     # → "positif"
print(résultat.score)         # → 0.92
print(résultat.mots_clés)     # → ["LangChain", "framework", "incroyable"]
print(résultat.résumé)        # → "Avis très positif sur LangChain..."
```

## Bind — attacher des paramètres fixes

```python
# bind() fixe des paramètres pour tous les appels de cette instance
llm_strict = llm.bind(
    temperature=0,      # toujours déterministe
    max_tokens=200,     # toujours court
    stop=["---"]        # s'arrête à ce token
)

# Équivalent à passer ces params à chaque invoke
chain = prompt | llm_strict | parser
```

## Fallbacks — modèle de secours

```python
from langchain_anthropic import ChatAnthropic
from langchain_openai import ChatOpenAI

llm_principal = ChatAnthropic(model="claude-sonnet-4-20250514")
llm_backup    = ChatOpenAI(model="gpt-4o-mini")

# Si le principal échoue → bascule automatiquement sur le backup
llm_avec_fallback = llm_principal.with_fallbacks([llm_backup])

réponse = llm_avec_fallback.invoke("Bonjour !")
# → Essaie Claude d'abord, GPT-4o-mini si Claude est down
```

## LiteLLM — interface universelle

Voir [[LiteLLM — Index des fiches]] pour un module dédié à `litellm.completion()` utilisée directement, sans passer par l'intégration LangChain.

```python
# Même code pour tous les LLM
from langchain_community.chat_models import ChatLiteLLM

llm_claude  = ChatLiteLLM(model="anthropic/claude-sonnet-4-20250514")
llm_gpt     = ChatLiteLLM(model="gpt-4o")
llm_mistral = ChatLiteLLM(model="mistral/mistral-large-latest")

# Changer de provider = changer uniquement la chaîne "model"
```

> [!tip] with_structured_output plutôt que JsonOutputParser
> Pour obtenir des sorties structurées, `with_structured_output(MonSchema)` est plus fiable que demander du JSON dans le prompt. Le LLM est contraint au schéma au niveau de l'API.

> [!warning] Ne pas confondre .invoke() et .stream()
> `.invoke()` attend la réponse complète avant de retourner. `.stream()` retourne les tokens au fur et à mesure. Pour une UI interactive, toujours utiliser `.stream()`.
