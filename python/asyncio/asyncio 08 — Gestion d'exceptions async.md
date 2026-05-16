#python #asyncio #exceptions #erreurs #try-except #exceptiongroup

## Exceptions dans les coroutines — comportement de base
```python
import asyncio

async def faulty() -> str:
    await asyncio.sleep(0.1)
    raise ValueError("Quelque chose s'est mal passé")

async def main() -> None:
    # Exception dans une coroutine awaitée directement → propagation normale
    try:
        await faulty()
    except ValueError as e:
        print(f"Capturé : {e}")   # ✅ fonctionne comme du code sync
```

## Exception dans une Task non awaitée — danger silencieux
```python
import asyncio

async def main() -> None:
    task = asyncio.create_task(faulty())
    # ❌ Si on n'awaite jamais task, l'exception est perdue !
    await asyncio.sleep(1)
    # → "Task exception was never retrieved" dans les logs — mais ne crash pas !

    # ✅ Toujours awaiter ou ajouter un done_callback
    task = asyncio.create_task(faulty())
    task.add_done_callback(
        lambda t: print(f"Erreur : {t.exception()}") if t.exception() else None
    )

    # ✅ Ou awaiter avec try/except
    task = asyncio.create_task(faulty())
    try:
        await task
    except ValueError as e:
        print(f"Task a échoué : {e}")
```

## gather — capturer toutes les exceptions
```python
import asyncio

async def might_fail(n: int) -> int:
    if n % 2 == 0:
        raise RuntimeError(f"Échec pour {n}")
    return n * 10

async def main() -> None:
    results = await asyncio.gather(
        *[might_fail(i) for i in range(5)],
        return_exceptions=True,   # ← capturer sans stopper
    )

    successes = [r for r in results if not isinstance(r, Exception)]
    failures  = [r for r in results if isinstance(r, Exception)]

    print(f"Succès : {successes}")   # [10, 30, 40]
    print(f"Échecs : {failures}")    # [RuntimeError(...), RuntimeError(...)]
```

## ExceptionGroup — TaskGroup Python 3.11+
```python
import asyncio

async def main() -> None:
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(faulty_a())
            tg.create_task(faulty_b())
            tg.create_task(ok_task())
    except* ValueError as eg:         # except* → ExceptionGroup
        print(f"ValueErrors : {eg.exceptions}")
    except* RuntimeError as eg:
        print(f"RuntimeErrors : {eg.exceptions}")
    # except* peut apparaître plusieurs fois — chaque branche traite un type
```

## Wrapper de tâche robuste
```python
import asyncio
import logging

logger = logging.getLogger(__name__)

async def safe_task(coro, name: str = ""):
    """Wrapper qui log les exceptions sans les perdre."""
    try:
        return await coro
    except asyncio.CancelledError:
        logger.info(f"Task {name!r} annulée")
        raise   # toujours re-raise CancelledError
    except Exception as e:
        logger.error(f"Task {name!r} a échoué : {e}", exc_info=True)
        return None   # ou raise selon le contexte

async def main() -> None:
    results = await asyncio.gather(
        safe_task(fetch("url1"), "fetch-1"),
        safe_task(fetch("url2"), "fetch-2"),
    )
```

## Cleanup avec try/finally dans les coroutines
```python
import asyncio

async def worker_with_cleanup() -> None:
    resource = await acquire_resource()
    try:
        await do_work(resource)
    except Exception as e:
        print(f"Erreur pendant le travail : {e}")
        raise
    finally:
        await resource.close()   # ✅ exécuté même en cas d'annulation ou d'exception
```

## asyncio.shield — protéger une coroutine de l'annulation
```python
import asyncio

async def critical_save(data: dict) -> None:
    """Ne doit pas être interrompu — sauvegarde critique."""
    await asyncio.sleep(2)   # simuler I/O critique
    print("Sauvegarde réussie")

async def main() -> None:
    task = asyncio.create_task(critical_save({"key": "value"}))

    await asyncio.sleep(0.5)
    task.cancel()   # annuler la task englobante

    try:
        # shield() protège critical_save même si task est annulée
        await asyncio.shield(critical_save({"key": "value"}))
    except asyncio.CancelledError:
        print("Annulation demandée, mais la sauvegarde continue en arrière-plan")
```

## Bonnes pratiques — résumé
```python
# 1. Toujours awaiter les tasks ou leur ajouter un done_callback
task = asyncio.create_task(coro())
await task  # ou task.add_done_callback(handler)

# 2. CancelledError → toujours re-raise
try:
    await something()
except asyncio.CancelledError:
    # cleanup...
    raise   # ← obligatoire

# 3. gather + return_exceptions=True pour les batches
results = await asyncio.gather(*coros, return_exceptions=True)

# 4. try/finally pour le cleanup des ressources
async with managed_resource() as r:
    await use(r)   # finally de __aexit__ garantit la fermeture

# 5. Log les exceptions non récupérées
asyncio.get_event_loop().set_exception_handler(my_handler)
```

> [!warning] "Task exception was never retrieved"
> Si une Task lève une exception et n'est jamais awaitée, Python log le message mais ne crash pas.
> C'est un bug silencieux — toujours awaiter les tasks ou inspecter leur résultat.
