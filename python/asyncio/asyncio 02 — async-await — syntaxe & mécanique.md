#python #asyncio #async #await #coroutine #syntaxe

## Syntaxe de base
```python
import asyncio

# async def → définit une coroutine
async def greet(name: str) -> str:
    await asyncio.sleep(0.1)   # await → suspend la coroutine
    return f"Bonjour {name}"

# await → attend le résultat d'une coroutine (ou d'un Awaitable)
async def main() -> None:
    result = await greet("Alice")
    print(result)   # "Bonjour Alice"

asyncio.run(main())
```

## Ce qu'on peut `await`
```python
# 1. Une coroutine
result = await some_coroutine()

# 2. Un asyncio.Task
task   = asyncio.create_task(some_coroutine())
result = await task

# 3. Un asyncio.Future
future = asyncio.get_event_loop().create_future()
result = await future

# 4. Tout objet avec __await__ (Awaitable)
from typing import Awaitable
```

## Mécanique — ce qui se passe à chaque `await`
```
async def fetch(url):
    print("1 — avant await")
    data = await http_get(url)    # ← SUSPENSION ICI
    print("3 — après await")      #   (2 = d'autres coroutines tournent)
    return data

Séquence :
  1. "1 — avant await"
  2. fetch() est suspendue → l'event loop reprend la main
  3. L'event loop exécute d'autres tâches prêtes
  4. Quand http_get() termine → fetch() est re-schedulée
  5. "3 — après await"
```

## async for — itération asynchrone
```python
import asyncio

# Un async iterable implémente __aiter__ et __anext__
async def ticker(n: int):
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async def main() -> None:
    async for value in ticker(5):
        print(value)   # 0, 1, 2, 3, 4 — avec 0.1s entre chaque

asyncio.run(main())
```

## async with — context managers asynchrones
```python
import asyncio
import aiofiles   # pip install aiofiles

async def read_file(path: str) -> str:
    async with aiofiles.open(path, "r") as f:   # __aenter__ / __aexit__
        return await f.read()

# Créer son propre async context manager
from contextlib import asynccontextmanager

@asynccontextmanager
async def managed_connection(url: str):
    conn = await connect(url)       # setup
    try:
        yield conn
    finally:
        await conn.close()           # teardown — toujours exécuté

async def main() -> None:
    async with managed_connection("postgresql://...") as conn:
        result = await conn.fetch("SELECT 1")
```

## Annotations de type pour les coroutines
```python
from collections.abc import AsyncIterator, AsyncGenerator, Coroutine
from typing import Awaitable

# Coroutine retournant un str
async def fetch(url: str) -> str: ...

# Fonction retournant une coroutine (non appelée)
def make_coro() -> Coroutine[None, None, str]: ...

# Async generator
async def stream() -> AsyncGenerator[int, None]:
    for i in range(10):
        yield i

# Async iterator
async def lines(path: str) -> AsyncIterator[str]: ...

# Awaitable générique
async def run(task: Awaitable[str]) -> str:
    return await task
```

## await n'est valide que dans async def
```python
# ❌ SyntaxError
def bad():
    await asyncio.sleep(1)   # await hors de async def

# ❌ Ne fait rien — crée un objet coroutine sans l'exécuter
def also_bad():
    asyncio.sleep(1)         # oubli de await — coroutine ignorée

# ✅
async def good():
    await asyncio.sleep(1)
```

> [!warning] Oublier `await` est silencieux mais catastrophique
> `asyncio.sleep(1)` sans `await` crée un objet coroutine et le jette immédiatement — aucun sleep, aucune erreur visible sauf le warning "coroutine was never awaited".
> Activer les warnings : `python -W error::RuntimeWarning script.py`
