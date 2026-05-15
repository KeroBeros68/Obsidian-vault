#python #decorateurs #typing #paramspec #typevar #générique

## Le problème — perte du typage avec Any
```python
from functools import wraps
from typing import Any, Callable

def log(func: Callable) -> Callable:   # ❌ retourne Callable sans info de type
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        print(f"Appel : {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def add(a: int, b: int) -> int:
    return a + b

reveal_type(add)        # Callable — mypy a perdu la signature ! ❌
add("x", "y")          # mypy ne détecte pas l'erreur ❌
```

## Solution — ParamSpec + TypeVar (Python 3.10+)
```python
from functools import wraps
from typing import TypeVar, ParamSpec
from collections.abc import Callable

P = ParamSpec("P")    # capture la signature des paramètres
R = TypeVar("R")      # capture le type de retour

def log(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print(f"Appel : {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def add(a: int, b: int) -> int:
    return a + b

reveal_type(add)    # (a: int, b: int) -> int ✅ mypy préserve la signature
add(1, 2)          # ✅
add("x", "y")      # ❌ mypy détecte l'erreur ✅
```

## Template complet — décorateur typé
```python
from functools import wraps
from typing import TypeVar, ParamSpec
from collections.abc import Callable

P = ParamSpec("P")
R = TypeVar("R")

def my_decorator(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        # code ici
        return func(*args, **kwargs)
    return wrapper
```

## Décorateur avec arguments — typé
```python
def repeat(times: int) -> Callable[[Callable[P, R]], Callable[P, R]]:
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result      # type: ignore[return-value]
        return wrapper
    return decorator

@repeat(3)
def hello(name: str) -> None:
    print(f"Bonjour {name}")

reveal_type(hello)   # (name: str) -> None ✅
```

## Décorateur asynchrone — Awaitable
```python
from collections.abc import Callable, Awaitable
from typing import TypeVar, ParamSpec
import asyncio

P = ParamSpec("P")
R = TypeVar("R")

def async_log(func: Callable[P, Awaitable[R]]) -> Callable[P, Awaitable[R]]:
    @wraps(func)
    async def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print(f"Async appel : {func.__name__}")
        result = await func(*args, **kwargs)
        print(f"Async retour : {func.__name__}")
        return result
    return wrapper

@async_log
async def fetch(url: str) -> dict:
    await asyncio.sleep(0.1)
    return {"url": url}
```

> [!tip] ParamSpec — Python 3.10+ ou typing_extensions
> Pour Python 3.9 et antérieur : `from typing_extensions import ParamSpec`
> Pour Python 3.10+ : `from typing import ParamSpec` directement
