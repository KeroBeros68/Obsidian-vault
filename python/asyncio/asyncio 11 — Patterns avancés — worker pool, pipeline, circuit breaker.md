#python #asyncio #patterns #worker-pool #pipeline #circuit-breaker

## Worker Pool — N workers sur une queue
```python
import asyncio
from collections.abc import Callable, Awaitable
from typing import TypeVar

T = TypeVar("T")
R = TypeVar("R")

async def worker_pool(
    items:      list[T],
    process:    Callable[[T], Awaitable[R]],
    n_workers:  int = 5,
) -> list[R | Exception]:
    """Lance n_workers traitant items en parallèle via une queue."""
    queue:   asyncio.Queue[T]              = asyncio.Queue()
    results: dict[int, R | Exception]      = {}

    for i, item in enumerate(items):
        await queue.put((i, item))

    async def worker() -> None:
        while not queue.empty():
            try:
                idx, item = queue.get_nowait()
            except asyncio.QueueEmpty:
                break
            try:
                results[idx] = await process(item)
            except Exception as e:
                results[idx] = e
            finally:
                queue.task_done()

    workers = [asyncio.create_task(worker()) for _ in range(n_workers)]
    await asyncio.gather(*workers)

    return [results[i] for i in range(len(items))]

# Usage
async def main() -> None:
    urls = [f"https://api.example.com/{i}" for i in range(20)]
    results = await worker_pool(urls, fetch, n_workers=5)
```

## Pipeline async — étapes enchaînées via queues
```python
import asyncio

async def stage(
    name:  str,
    inbox: asyncio.Queue,
    outbox: asyncio.Queue,
    process: Callable,
) -> None:
    while True:
        item = await inbox.get()
        if item is None:          # sentinel
            await outbox.put(None)
            inbox.task_done()
            break
        result = await process(item)
        await outbox.put(result)
        inbox.task_done()

async def pipeline_demo() -> None:
    q1: asyncio.Queue = asyncio.Queue()
    q2: asyncio.Queue = asyncio.Queue()
    q3: asyncio.Queue = asyncio.Queue()

    async def double(x): return x * 2
    async def to_str(x): return f"item-{x}"

    s1 = asyncio.create_task(stage("double", q1, q2, double))
    s2 = asyncio.create_task(stage("str",    q2, q3, to_str))

    # Injecter les données
    for i in range(10):
        await q1.put(i)
    await q1.put(None)   # signal de fin

    await asyncio.gather(s1, s2)

    # Lire les résultats
    results = []
    while not q3.empty():
        item = q3.get_nowait()
        if item is not None:
            results.append(item)
    print(results)
```

## Circuit Breaker async
```python
import asyncio
import time
from enum import Enum

class State(Enum):
    CLOSED  = "closed"    # normal — laisse passer
    OPEN    = "open"      # en défaut — bloque tout
    HALF    = "half"      # test — laisse passer un essai

class AsyncCircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=30.0):
        self.failure_threshold  = failure_threshold
        self.recovery_timeout   = recovery_timeout
        self.state              = State.CLOSED
        self.failure_count      = 0
        self.last_failure_time  = 0.0

    async def call(self, coro):
        if self.state == State.OPEN:
            if time.monotonic() - self.last_failure_time > self.recovery_timeout:
                self.state = State.HALF
            else:
                raise RuntimeError("Circuit ouvert — service indisponible")

        try:
            result = await coro
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        self.failure_count = 0
        self.state         = State.CLOSED

    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.monotonic()
        if self.failure_count >= self.failure_threshold:
            self.state = State.OPEN

cb = AsyncCircuitBreaker(failure_threshold=3, recovery_timeout=10.0)

async def safe_fetch(url: str) -> str:
    return await cb.call(fetch(url))
```

## Debounce & Throttle async
```python
import asyncio
import time

def async_debounce(func, delay: float):
    """N'exécute func que si delay secondes se sont écoulées sans nouvel appel."""
    _task: asyncio.Task | None = None

    async def debounced(*args, **kwargs):
        nonlocal _task
        if _task:
            _task.cancel()
        async def _run():
            await asyncio.sleep(delay)
            await func(*args, **kwargs)
        _task = asyncio.create_task(_run())

    return debounced

def async_throttle(func, interval: float):
    """N'exécute func qu'une fois par interval secondes maximum."""
    _last_call = 0.0

    async def throttled(*args, **kwargs):
        nonlocal _last_call
        now = time.monotonic()
        if now - _last_call >= interval:
            _last_call = now
            return await func(*args, **kwargs)

    return throttled
```

## Mutex avec timeout
```python
import asyncio

async def with_lock_timeout(lock: asyncio.Lock, timeout: float):
    """Acquérir un lock avec timeout."""
    try:
        async with asyncio.timeout(timeout):
            async with lock:
                yield
    except TimeoutError:
        raise RuntimeError(f"Impossible d'acquérir le lock en {timeout}s")
```
