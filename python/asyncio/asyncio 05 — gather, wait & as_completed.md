#python #asyncio #gather #wait #as_completed #concurrence

## asyncio.gather — lancer N coroutines en parallèle
```python
import asyncio

async def fetch(url: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"data:{url}"

async def main() -> None:
    # gather : lance tout en parallèle, attend tout, retourne dans l'ordre d'entrée
    results = await asyncio.gather(
        fetch("url1", 1.0),
        fetch("url2", 2.0),
        fetch("url3", 0.5),
    )
    # Durée totale ≈ 2s (le plus lent), pas 3.5s (somme)
    print(results)   # ["data:url1", "data:url2", "data:url3"]
    # ↑ ordre préservé — même si url3 a fini en premier !
```

## gather — gestion des exceptions
```python
import asyncio

async def might_fail(n: int) -> int:
    if n == 2:
        raise ValueError(f"Erreur sur {n}")
    await asyncio.sleep(0.1)
    return n * 10

async def main() -> None:
    # Par défaut : première exception → annule tout + re-raise
    try:
        results = await asyncio.gather(
            might_fail(1),
            might_fail(2),   # ← lève ValueError
            might_fail(3),
        )
    except ValueError as e:
        print(f"Erreur : {e}")   # les autres tasks sont annulées

    # return_exceptions=True : capturer les exceptions comme résultats
    results = await asyncio.gather(
        might_fail(1),
        might_fail(2),
        might_fail(3),
        return_exceptions=True,
    )
    for i, r in enumerate(results):
        if isinstance(r, Exception):
            print(f"Task {i} a échoué : {r}")
        else:
            print(f"Task {i} : {r}")
    # Task 0 : 10
    # Task 1 a échoué : Erreur sur 2
    # Task 2 : 30
```

## asyncio.wait — attendre avec contrôle fin
```python
import asyncio

async def main() -> None:
    tasks = [asyncio.create_task(fetch(url)) for url in urls]

    # FIRST_COMPLETED — dès qu'une task finit
    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED
    )
    for t in done:
        print(t.result())
    for t in pending:
        t.cancel()   # annuler le reste

    # FIRST_EXCEPTION — dès qu'une exception
    done, pending = await asyncio.wait(tasks, return_when=asyncio.FIRST_EXCEPTION)

    # ALL_COMPLETED (défaut) — attendre tout
    done, pending = await asyncio.wait(tasks)   # pending = {} à la fin

    # Avec timeout
    done, pending = await asyncio.wait(tasks, timeout=5.0)
    # pending contient les tasks non terminées après 5s
```

## asyncio.as_completed — traiter au fil de l'eau
```python
import asyncio

async def main() -> None:
    coros = [fetch(f"url{i}", delay=i*0.3) for i in range(5)]

    # as_completed retourne les futures dans l'ordre de completion (pas d'entrée)
    for coro in asyncio.as_completed(coros):
        result = await coro
        print(f"Reçu : {result}")   # url0 d'abord (0.0s), puis url1 (0.3s)...
    # ← traitement progressif — sans attendre que tout soit fini

    # Avec timeout global
    for coro in asyncio.as_completed(coros, timeout=2.0):
        try:
            result = await coro
        except asyncio.TimeoutError:
            print("Une tâche a dépassé le timeout global")
```

## Comparaison — gather vs wait vs as_completed
| | `gather` | `wait` | `as_completed` |
|---|---|---|---|
| Ordre de retour | Ordre entrée | Set (done/pending) | Ordre completion |
| Exception | Stoppe tout (défaut) | Configurable | Par tâche |
| Traitement progressif | ❌ | ❌ | ✅ |
| Annulation partielle | ❌ natif | ✅ manuel | ❌ natif |
| Timeout | ❌ natif | ✅ | ✅ |
| Syntaxe | Simple | Verbose | Intermédiaire |

## Patterns courants

**Fan-out / Fan-in — lancer N requêtes, attendre toutes**
```python
async def fan_out(urls: list[str]) -> list[str]:
    return await asyncio.gather(*[fetch(url) for url in urls])
```

**Premier résultat valide**
```python
async def first_valid(coros) -> str | None:
    for coro in asyncio.as_completed(coros):
        result = await coro
        if result is not None:
            return result
    return None
```

**Limiter la concurrence avec Semaphore**
```python
async def limited_gather(urls: list[str], max_concurrent: int = 5) -> list[str]:
    sem = asyncio.Semaphore(max_concurrent)
    async def _fetch(url: str) -> str:
        async with sem:
            return await fetch(url)
    return await asyncio.gather(*[_fetch(url) for url in urls])
```

> [!tip] gather avec return_exceptions=True — le meilleur compromis
> Pour les pipelines robustes, `gather(*coros, return_exceptions=True)` capture tout sans crasher.
> Inspecter chaque résultat avec `isinstance(r, Exception)`.
