#python #asyncio #lock #event #semaphore #synchronisation

## asyncio.Lock — exclusion mutuelle
```python
import asyncio

# Lock empêche deux coroutines d'accéder simultanément à une ressource partagée
lock = asyncio.Lock()

shared_resource = []

async def safe_append(item: str) -> None:
    async with lock:           # acquire + release automatiques
        # Section critique — une seule coroutine à la fois ici
        shared_resource.append(item)
        await asyncio.sleep(0)   # même avec un await ici, le lock est tenu

async def main() -> None:
    await asyncio.gather(
        safe_append("A"),
        safe_append("B"),
        safe_append("C"),
    )
    print(shared_resource)   # ["A", "B", "C"] — pas de race condition

# Acquisition manuelle
await lock.acquire()
try:
    # section critique
finally:
    lock.release()
```

## asyncio.Event — signal entre coroutines
```python
import asyncio

event = asyncio.Event()

async def waiter(name: str) -> None:
    print(f"{name} attend l'événement...")
    await event.wait()     # bloque jusqu'à event.set()
    print(f"{name} a reçu le signal !")

async def trigger() -> None:
    await asyncio.sleep(2)
    print("Déclenchement !")
    event.set()            # réveille TOUS les waiters

async def main() -> None:
    await asyncio.gather(
        waiter("W1"),
        waiter("W2"),
        waiter("W3"),
        trigger(),
    )
    # Après set() : event reste True
    event.clear()          # remettre à False pour réutiliser
    event.is_set()         # True / False
```

## asyncio.Semaphore — limiter la concurrence
```python
import asyncio

# Semaphore : au plus N coroutines simultanées dans la section critique
sem = asyncio.Semaphore(3)   # max 3 requêtes simultanées

async def limited_fetch(session, url: str) -> str:
    async with sem:            # acquire si < 3 actifs, sinon attend
        return await session.get(url)

async def main() -> None:
    urls = [f"https://api.example.com/{i}" for i in range(20)]
    results = await asyncio.gather(
        *[limited_fetch(session, url) for url in urls]
    )
    # Bien que 20 tâches soient créées, au plus 3 tournent simultanément

# BoundedSemaphore — lève ValueError si release() sans acquire()
sem = asyncio.BoundedSemaphore(3)
```

## asyncio.Condition — Lock + Event combinés
```python
import asyncio

condition = asyncio.Condition()
shared_data = []

async def consumer() -> None:
    async with condition:
        while not shared_data:
            await condition.wait()   # libère le lock et attend
        item = shared_data.pop()
        print(f"Consommé : {item}")

async def producer() -> None:
    async with condition:
        shared_data.append("item")
        condition.notify()           # réveille un waiter
        # condition.notify_all()     # réveille tous les waiters
```

## asyncio.Barrier — synchronisation par lot (Python 3.11+)
```python
import asyncio

barrier = asyncio.Barrier(3)   # attend que 3 coroutines arrivent

async def worker(n: int) -> None:
    print(f"Worker {n} arrive à la barrière")
    await barrier.wait()       # bloque jusqu'à ce que 3 soient arrivés
    print(f"Worker {n} franchit la barrière")

async def main() -> None:
    await asyncio.gather(worker(1), worker(2), worker(3))
    # Tous les workers se "synchronisent" au même point
```

## Tableau de synthèse
| Primitive | Cas d'usage | Clé |
|---|---|---|
| `Lock` | Section critique — 1 seul à la fois | `async with lock` |
| `Event` | Signal one-shot entre coroutines | `event.set()` / `event.wait()` |
| `Semaphore` | Limiter la concurrence à N | `async with sem` |
| `BoundedSemaphore` | Semaphore avec vérification d'abus | idem |
| `Condition` | Wait sur une condition + notification | `cond.wait()` / `cond.notify()` |
| `Barrier` | Synchronisation par groupe (3.11+) | `barrier.wait()` |
| `Queue` | Producteur/consommateur | `put()` / `get()` |

> [!tip] asyncio.Lock vs threading.Lock
> `asyncio.Lock` n'est **pas thread-safe** — à n'utiliser que dans des coroutines.
> Pour la communication entre threads et coroutines : `asyncio.get_event_loop().call_soon_threadsafe()` ou `loop.run_in_executor()`.
