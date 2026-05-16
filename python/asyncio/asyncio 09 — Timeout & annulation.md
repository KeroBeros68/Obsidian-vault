#python #asyncio #timeout #cancel #annulation #asyncio-timeout

## asyncio.timeout — gestionnaire de contexte (Python 3.11+)
```python
import asyncio

async def main() -> None:
    # ✅ Méthode recommandée en Python 3.11+
    try:
        async with asyncio.timeout(5.0):
            result = await long_operation()
    except TimeoutError:
        print("Opération trop longue — abandonnée")

    # Avec deadline absolue
    deadline = asyncio.get_event_loop().time() + 10.0
    async with asyncio.timeout_at(deadline):
        result = await operation()
```

## asyncio.wait_for — timeout sur une coroutine (compatible 3.8+)
```python
import asyncio

async def main() -> None:
    # wait_for : timeout sur une coroutine ou une task
    try:
        result = await asyncio.wait_for(
            long_operation(),
            timeout=5.0,
        )
    except asyncio.TimeoutError:
        print("Timeout !")
        # La coroutine long_operation est annulée automatiquement

    # Avec une Task existante
    task = asyncio.create_task(long_operation())
    try:
        result = await asyncio.wait_for(task, timeout=5.0)
    except asyncio.TimeoutError:
        print("Task annulée par timeout")
        # task.cancelled() → True
```

## Timeout avec retry
```python
import asyncio

async def fetch_with_retry(url: str, retries: int = 3, timeout: float = 5.0) -> str:
    last_err: Exception | None = None
    for attempt in range(1, retries + 1):
        try:
            return await asyncio.wait_for(fetch(url), timeout=timeout)
        except asyncio.TimeoutError:
            last_err = asyncio.TimeoutError(f"Timeout sur {url} (tentative {attempt})")
            print(f"Tentative {attempt}/{retries} — timeout")
            await asyncio.sleep(2 ** attempt)   # backoff exponentiel
        except Exception as e:
            last_err = e
            print(f"Tentative {attempt}/{retries} — erreur : {e}")
            await asyncio.sleep(1)
    raise last_err
```

## Annulation propre — pattern complet
```python
import asyncio
import signal

async def worker() -> None:
    try:
        while True:
            await asyncio.sleep(1)
            print("Travaille...")
    except asyncio.CancelledError:
        print("Worker : nettoyage en cours...")
        await asyncio.sleep(0.1)   # cleanup async possible ici
        print("Worker : nettoyage terminé")
        raise   # re-raise obligatoire

async def main() -> None:
    task = asyncio.create_task(worker())

    # Shutdown propre sur SIGTERM / SIGINT
    loop = asyncio.get_running_loop()

    def shutdown():
        print("Signal reçu — arrêt...")
        task.cancel()

    loop.add_signal_handler(signal.SIGTERM, shutdown)
    loop.add_signal_handler(signal.SIGINT,  shutdown)

    try:
        await task
    except asyncio.CancelledError:
        pass   # attendu
```

## Annuler un groupe de tasks proprement
```python
import asyncio

async def shutdown_tasks(tasks: list[asyncio.Task]) -> None:
    """Annuler toutes les tasks et attendre leur terminaison."""
    for task in tasks:
        if not task.done():
            task.cancel()

    results = await asyncio.gather(*tasks, return_exceptions=True)

    for task, result in zip(tasks, results):
        if isinstance(result, asyncio.CancelledError):
            pass   # normal
        elif isinstance(result, Exception):
            print(f"Task {task.get_name()} a échoué : {result}")
```

## Comparaison — méthodes de timeout
| Méthode | Python | Annulation auto | Style |
|---|---|---|---|
| `asyncio.timeout()` | 3.11+ | ✅ | `async with` |
| `asyncio.timeout_at()` | 3.11+ | ✅ | `async with` |
| `asyncio.wait_for()` | 3.7+ | ✅ | `await` |
| `asyncio.wait(..., timeout=)` | 3.7+ | ❌ (pending non annulé) | `await` |
| `as_completed(..., timeout=)` | 3.7+ | ❌ | `for` |

> [!warning] asyncio.wait() avec timeout ne cancel pas les pending
> Contrairement à `wait_for`, `asyncio.wait(tasks, timeout=5)` laisse les tasks non terminées en vie.
> Il faut annuler manuellement les tasks dans le set `pending` retourné.
