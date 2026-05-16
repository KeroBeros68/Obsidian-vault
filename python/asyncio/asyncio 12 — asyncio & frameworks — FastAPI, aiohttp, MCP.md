#python #asyncio #fastapi #aiohttp #mcp #frameworks

## FastAPI — ASGI & routes async
```python
from fastapi import FastAPI
import asyncio
import httpx   # client HTTP async

app = FastAPI()

# Route async — gère des milliers de requêtes concurrent
@app.get("/users/{user_id}")
async def get_user(user_id: int) -> dict:
    # Ne bloque pas — d'autres requêtes peuvent tourner pendant l'await
    user = await db.fetch_one("SELECT * FROM users WHERE id = $1", user_id)
    return {"id": user["id"], "name": user["name"]}

# Appels concurrents dans une route
@app.get("/dashboard")
async def dashboard(user_id: int) -> dict:
    user, orders, notifications = await asyncio.gather(
        fetch_user(user_id),
        fetch_orders(user_id),
        fetch_notifications(user_id),
    )
    return {"user": user, "orders": orders, "notifications": notifications}

# Lifespan — startup / shutdown async
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await db.connect()
    yield
    # Shutdown
    await db.disconnect()

app = FastAPI(lifespan=lifespan)
```

## aiohttp — client HTTP async
```python
import asyncio
import aiohttp   # pip install aiohttp

async def fetch_all(urls: list[str]) -> list[str]:
    async with aiohttp.ClientSession() as session:
        async def fetch_one(url: str) -> str:
            async with session.get(url) as response:
                response.raise_for_status()
                return await response.text()

        return await asyncio.gather(*[fetch_one(url) for url in urls])

# Avec timeout et retry
async def robust_fetch(url: str) -> str:
    timeout = aiohttp.ClientTimeout(total=30, connect=5)
    async with aiohttp.ClientSession(timeout=timeout) as session:
        for attempt in range(3):
            try:
                async with session.get(url) as resp:
                    return await resp.json()
            except (aiohttp.ClientError, asyncio.TimeoutError) as e:
                if attempt == 2: raise
                await asyncio.sleep(2 ** attempt)
```

## MCP Client async — pattern complet
```python
# → [[MCP 04 — Client Python async]]
import asyncio
from mcp import ClientSession
from mcp.client.stdio import stdio_client

async def mcp_client_example() -> None:
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Lister les outils disponibles
            tools = await session.list_tools()

            # Appeler un outil
            result = await session.call_tool(
                "search",
                arguments={"query": "asyncio python"}
            )

            # Lire une ressource
            resource = await session.read_resource("file:///data.json")

async def main() -> None:
    await mcp_client_example()

asyncio.run(main())
```

## LangChain / LangGraph — agents async
```python
import asyncio
from langchain_core.runnables import RunnableSequence

# Les chains LangChain supportent ainvoke / astream / abatch
async def run_agent(query: str) -> str:
    chain: RunnableSequence = prompt | llm | parser

    # Invocation async unique
    result = await chain.ainvoke({"question": query})

    # Streaming async
    async for chunk in chain.astream({"question": query}):
        print(chunk, end="", flush=True)

    # Batch async — plusieurs requêtes en parallèle
    results = await chain.abatch([
        {"question": "Q1"},
        {"question": "Q2"},
        {"question": "Q3"},
    ])
    return result

asyncio.run(run_agent("Qu'est-ce que asyncio ?"))
```

## asyncpg / databases — BDD async
```python
import asyncio
import asyncpg   # pip install asyncpg

async def db_example() -> None:
    # Connection pool
    pool = await asyncpg.create_pool(
        "postgresql://user:pass@localhost/db",
        min_size=5, max_size=20,
    )
    async with pool.acquire() as conn:
        # Requêtes concurrentes sur le pool
        users = await conn.fetch("SELECT * FROM users")
        await conn.execute("UPDATE users SET active=$1 WHERE id=$2", True, 1)

    # Transaction async
    async with pool.acquire() as conn:
        async with conn.transaction():
            await conn.execute("INSERT INTO orders VALUES ($1, $2)", 1, "item")
            await conn.execute("UPDATE stock SET qty = qty - 1 WHERE id = $1", 1)

    await pool.close()
```

## Checklist asyncio en production
```
✅ asyncio.run(main()) comme unique point d'entrée
✅ Toutes les coroutines awaitées (ou callbacks ajoutés)
✅ CancelledError toujours re-raised
✅ try/finally pour le cleanup des ressources
✅ Semaphore pour limiter la concurrence réseau
✅ asyncio.to_thread() pour les librairies synchrones
✅ mode debug en développement (PYTHONASYNCIODEBUG=1)
✅ Connection pools pour BDD (ne pas ouvrir une connexion par requête)
✅ Timeouts sur toutes les opérations réseau
✅ Lifespan FastAPI pour gérer startup/shutdown proprement
```
