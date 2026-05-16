#python #asyncio #queue #producteur #consommateur #backpressure

## asyncio.Queue — file thread-safe pour coroutines
```python
import asyncio

async def main() -> None:
    queue: asyncio.Queue[str] = asyncio.Queue(maxsize=10)

    # Producteur — ajouter des éléments
    await queue.put("item1")           # bloque si queue pleine (maxsize atteint)
    queue.put_nowait("item2")          # lève QueueFull si pleine

    # Consommateur — retirer des éléments
    item = await queue.get()           # bloque si queue vide
    item = queue.get_nowait()          # lève QueueEmpty si vide

    # Signaler qu'un item a été traité (pour join())
    queue.task_done()

    # Attendre que tous les items soient traités
    await queue.join()

    # État
    queue.qsize()     # nombre d'items actuellement dans la queue
    queue.empty()     # True si vide
    queue.full()      # True si pleine (maxsize atteint)
```

## Pattern producteur / consommateur
```python
import asyncio
from typing import Any

async def producer(queue: asyncio.Queue[str], items: list[str]) -> None:
    for item in items:
        await queue.put(item)
        print(f"Produit : {item}")
        await asyncio.sleep(0.1)   # simuler un délai de production

async def consumer(queue: asyncio.Queue[str], worker_id: int) -> None:
    while True:
        item = await queue.get()   # attend si vide
        try:
            print(f"[Worker {worker_id}] Traite : {item}")
            await asyncio.sleep(0.3)   # simuler le traitement
        finally:
            queue.task_done()          # toujours appeler même si exception

async def main() -> None:
    queue: asyncio.Queue[str] = asyncio.Queue(maxsize=5)
    items = [f"item-{i}" for i in range(10)]

    # Lancer 1 producteur et 3 consommateurs
    producer_task = asyncio.create_task(producer(queue, items))
    consumers = [
        asyncio.create_task(consumer(queue, i))
        for i in range(3)
    ]

    await producer_task      # attendre que tout soit produit
    await queue.join()       # attendre que tout soit consommé

    # Arrêter les consommateurs (ils boucleraient indéfiniment)
    for c in consumers:
        c.cancel()
    await asyncio.gather(*consumers, return_exceptions=True)
```

## Sentinel — signal d'arrêt propre
```python
import asyncio
from typing import Optional

STOP = object()   # objet sentinelle unique

async def producer(queue: asyncio.Queue, n_workers: int) -> None:
    for i in range(20):
        await queue.put(f"task-{i}")
    # Envoyer un signal d'arrêt par worker
    for _ in range(n_workers):
        await queue.put(STOP)

async def consumer(queue: asyncio.Queue, worker_id: int) -> None:
    while True:
        item = await queue.get()
        if item is STOP:
            queue.task_done()
            break              # sortie propre
        # traiter item...
        print(f"[{worker_id}] {item}")
        queue.task_done()
```

## Variantes — PriorityQueue et LifoQueue
```python
import asyncio

# File de priorité — items triés par (priorité, valeur)
pq: asyncio.PriorityQueue[tuple[int, str]] = asyncio.PriorityQueue()
await pq.put((1, "urgent"))
await pq.put((3, "basse priorité"))
await pq.put((2, "normale"))

_, item = await pq.get()   # (1, "urgent") — la plus haute priorité (valeur min)

# Pile LIFO async
lq: asyncio.LifoQueue[str] = asyncio.LifoQueue()
await lq.put("premier")
await lq.put("deuxième")
await lq.get()   # "deuxième"
```

## Backpressure — contrôler la vitesse de production
```python
import asyncio

async def controlled_producer(queue: asyncio.Queue[str]) -> None:
    for i in range(100):
        # queue.put() BLOQUE si maxsize atteint → backpressure naturelle
        await queue.put(f"item-{i}")
        # Le producteur ne peut pas dépasser la capacité de consommation

async def slow_consumer(queue: asyncio.Queue[str]) -> None:
    while True:
        item = await queue.get()
        await asyncio.sleep(1)   # traitement lent
        queue.task_done()
        print(f"Traité : {item}")

async def main() -> None:
    # maxsize=3 → le producteur est limité à 3 items d'avance
    queue: asyncio.Queue[str] = asyncio.Queue(maxsize=3)
    await asyncio.gather(
        controlled_producer(queue),
        slow_consumer(queue),
    )
```

> [!tip] queue.task_done() dans finally
> Toujours appeler `queue.task_done()` dans un bloc `finally` pour garantir l'appel même si le traitement lève une exception — sinon `queue.join()` ne se terminera jamais.
