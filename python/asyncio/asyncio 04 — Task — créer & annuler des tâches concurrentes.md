#python #asyncio #task #create_task #concurrence #annulation

## Task — exécution concurrente
```python
import asyncio

async def fetch(url: str) -> str:
    await asyncio.sleep(1)
    return f"data from {url}"

async def main() -> None:
    # await seul → séquentiel (1s + 1s = 2s)
    r1 = await fetch("url1")
    r2 = await fetch("url2")

    # create_task → concurrent (max(1s, 1s) = 1s)
    task1 = asyncio.create_task(fetch("url1"))
    task2 = asyncio.create_task(fetch("url2"))
    r1 = await task1
    r2 = await task2

asyncio.run(main())
```

## asyncio.create_task — planifier immédiatement
```python
import asyncio

async def worker(name: str, delay: float) -> str:
    print(f"{name} démarré")
    await asyncio.sleep(delay)
    print(f"{name} terminé")
    return f"résultat {name}"

async def main() -> None:
    # create_task planifie la coroutine sur la loop courante immédiatement
    task_a = asyncio.create_task(worker("A", 2.0), name="worker-A")
    task_b = asyncio.create_task(worker("B", 1.0), name="worker-B")

    # B termine avant A (1s < 2s)
    result_b = await task_b   # "résultat B"
    result_a = await task_a   # "résultat A"
    print(task_a.get_name())  # "worker-A"
```

## Task — états et méthodes
```python
task = asyncio.create_task(some_coroutine())

# État
task.done()       # True si terminée (succès, exception, ou annulée)
task.cancelled()  # True si annulée
task.result()     # retourne le résultat (lève l'exception si exception, ou CancelledError)
task.exception()  # retourne l'exception si elle a eu lieu (None sinon)

# Nom
task.get_name()        # "Task-1" par défaut
task.set_name("mon-nom")

# Callbacks — appelés quand la task se termine
task.add_done_callback(lambda t: print(f"Task finie : {t.result()}"))
```

## Annulation d'une Task
```python
import asyncio

async def long_job() -> str:
    try:
        print("Démarré")
        await asyncio.sleep(10)
        return "terminé"
    except asyncio.CancelledError:
        print("Annulé proprement — nettoyage...")
        raise   # ← toujours re-raise CancelledError !

async def main() -> None:
    task = asyncio.create_task(long_job())

    await asyncio.sleep(1)   # laisser tourner 1 seconde
    task.cancel()            # envoie CancelledError à la coroutine

    try:
        await task           # attendre la fin (levée de CancelledError)
    except asyncio.CancelledError:
        print("Task annulée confirmée")
```

## asyncio.current_task & all_tasks
```python
async def show_tasks() -> None:
    me = asyncio.current_task()        # la Task courante
    all_tasks = asyncio.all_tasks()    # toutes les Tasks de la loop (sauf current)

    for t in all_tasks:
        print(t.get_name(), t.done())
```

## TaskGroup — gestion groupée (Python 3.11+)
```python
import asyncio

async def main() -> None:
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch("url1"))
        task2 = tg.create_task(fetch("url2"))
        task3 = tg.create_task(fetch("url3"))
    # ← on arrive ici quand TOUTES les tasks sont terminées
    # Si une task lève une exception → toutes les autres sont annulées
    # → ExceptionGroup levée avec toutes les exceptions

    print(task1.result())
    print(task2.result())
```

## Task vs Future
| | `Task` | `Future` |
|---|---|---|
| Créé par | `create_task(coro)` | `loop.create_future()` |
| Contenu | wraps une coroutine | résultat positionné manuellement |
| Auto-planifié | ✅ immédiatement | ❌ rempli explicitement |
| Usage | concurrence de coroutines | primitives bas niveau, callbacks |

> [!warning] Toujours re-raise CancelledError
> Une coroutine qui attrape `CancelledError` doit toujours la re-lancer (`raise`) après nettoyage. Si elle l'avale, la task ne sera jamais marquée comme annulée → deadlock possible.

> [!tip] create_task vs ensure_future
> `asyncio.ensure_future()` est l'ancienne API (Python < 3.7) — accepte coroutines ET futures.
> En Python 3.7+, utiliser toujours `asyncio.create_task()` pour les coroutines.
