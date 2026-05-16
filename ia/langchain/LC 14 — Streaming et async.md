#ia #langchain #streaming #async #production #avancé

## Streaming et async

## Streaming — afficher les tokens au fur et à mesure

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm    = ChatAnthropic(model="claude-sonnet-4-20250514")
prompt = ChatPromptTemplate.from_messages([("human", "{question}")])
chain  = prompt | llm | StrOutputParser()

# Streaming basique — affiche token par token
for chunk in chain.stream({"question": "Explique le RAG en 5 points"}):
    print(chunk, end="", flush=True)
print()  # saut de ligne final

# Streaming avec accumulation
réponse_complète = ""
for chunk in chain.stream({"question": "Bonjour !"}):
    réponse_complète += chunk
    print(chunk, end="", flush=True)
print(f"\nRéponse complète : {réponse_complète}")
```

## astream() — streaming asynchrone

```python
import asyncio
from langchain_anthropic import ChatAnthropic

llm   = ChatAnthropic(model="claude-sonnet-4-20250514")
chain = prompt | llm | StrOutputParser()

async def streamer_async():
    async for chunk in chain.astream({"question": "Bonjour !"}):
        print(chunk, end="", flush=True)

asyncio.run(streamer_async())
```

## astream_events() — événements détaillés

Utile pour distinguer les tokens LLM des résultats d'outils dans un agent.

```python
async def streamer_avec_events():
    async for event in agent.astream_events(
        {"messages": [{"role": "user", "content": "Calcule 15 * 8"}]},
        version="v2"
    ):
        kind = event["event"]

        if kind == "on_chat_model_stream":
            # Token LLM
            chunk = event["data"]["chunk"]
            print(chunk.content, end="", flush=True)

        elif kind == "on_tool_start":
            # Outil appelé
            print(f"\n[Tool: {event['name']} → {event['data']['input']}]")

        elif kind == "on_tool_end":
            # Résultat de l'outil
            print(f"[Résultat: {event['data']['output']}]")

asyncio.run(streamer_avec_events())
```

## batch() — traitement parallèle

```python
# Traiter plusieurs inputs simultanément (parallèle)
questions = [
    {"question": "Capitale de France ?"},
    {"question": "Capitale d'Allemagne ?"},
    {"question": "Capitale d'Italie ?"}
]

# Appels en parallèle — beaucoup plus rapide qu'une boucle for
réponses = chain.batch(questions, config={"max_concurrency": 3})

for i, r in enumerate(réponses):
    print(f"Q{i+1}: {r}")
```

## Async complet — FastAPI + streaming

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

app_api = FastAPI()
llm     = ChatAnthropic(model="claude-sonnet-4-20250514")
prompt  = ChatPromptTemplate.from_messages([("human", "{question}")])
chain   = prompt | llm | StrOutputParser()

@app_api.get("/chat")
async def chat(question: str):
    async def générer():
        async for chunk in chain.astream({"question": question}):
            yield f"data: {chunk}\n\n"   # format Server-Sent Events
        yield "data: [DONE]\n\n"

    return StreamingResponse(générer(), media_type="text/event-stream")

# Lancer : uvicorn main:app_api --reload
```

## abatch() — batch asynchrone

```python
import asyncio

async def batch_async():
    questions = [{"question": f"Question {i}"} for i in range(10)]
    réponses = await chain.abatch(
        questions,
        config={"max_concurrency": 5}   # 5 appels simultanés max
    )
    return réponses

résultats = asyncio.run(batch_async())
```

> [!tip] stream() pour les UIs, batch() pour les pipelines
> Utilise `.stream()` ou `.astream()` pour les interfaces utilisateur (réponse progressive). Utilise `.batch()` ou `.abatch()` pour les traitements en lot (enrichissement de données, génération en masse).

> [!warning] flush=True indispensable pour le streaming
> Sans `flush=True`, Python peut bufferiser les sorties et afficher tout d'un coup à la fin. Toujours `print(chunk, end="", flush=True)` pour un vrai streaming.
