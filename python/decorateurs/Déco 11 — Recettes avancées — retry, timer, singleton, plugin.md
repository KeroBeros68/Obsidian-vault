#python #decorateurs #recettes #patterns #avancé

## Retry avec backoff exponentiel
```python
import time
import random
from functools import wraps

def retry(
    max_attempts: int = 3,
    delay: float = 1.0,
    backoff: float = 2.0,
    jitter: bool = True,
    exceptions: tuple[type[Exception], ...] = (Exception,),
):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            current_delay = delay
            last_exc: Exception | None = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exc = e
                    if attempt == max_attempts:
                        break
                    wait = current_delay + (random.random() * 0.5 if jitter else 0)
                    print(f"Tentative {attempt}/{max_attempts} — attente {wait:.2f}s")
                    time.sleep(wait)
                    current_delay *= backoff   # backoff exponentiel
            raise last_exc
        return wrapper
    return decorator

@retry(max_attempts=4, delay=0.5, exceptions=(ConnectionError, TimeoutError))
def call_api(url: str) -> dict:
    ...
```

## Validateur de types à l'exécution — runtime type check
```python
import inspect
from functools import wraps
from typing import get_type_hints

def validate_types(func):
    """Valide les types des arguments au runtime selon les annotations."""
    hints = get_type_hints(func)
    sig   = inspect.signature(func)

    @wraps(func)
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        for param_name, value in bound.arguments.items():
            if param_name in hints and param_name != "return":
                expected = hints[param_name]
                if not isinstance(value, expected):
                    raise TypeError(
                        f"{func.__name__}: {param_name} doit être {expected.__name__}, "
                        f"reçu {type(value).__name__}"
                    )
        result = func(*args, **kwargs)
        if "return" in hints and hints["return"] is not type(None):
            if not isinstance(result, hints["return"]):
                raise TypeError(f"Retour attendu {hints['return'].__name__}")
        return result
    return wrapper

@validate_types
def add(a: int, b: int) -> int:
    return a + b

add(1, 2)        # ✅
add("x", 2)      # ❌ TypeError: add: a doit être int, reçu str
```

## Registre de plugins — décorateur comme enregistreur
```python
from typing import Callable

class PluginRegistry:
    """Registre de plugins déclarés avec un décorateur."""
    def __init__(self) -> None:
        self._plugins: dict[str, Callable] = {}

    def register(self, name: str | None = None):
        def decorator(func: Callable) -> Callable:
            key = name or func.__name__
            self._plugins[key] = func
            return func
        return decorator

    def get(self, name: str) -> Callable:
        return self._plugins[name]

    def list(self) -> list[str]:
        return list(self._plugins.keys())

registry = PluginRegistry()

@registry.register()
def csv_loader(path: str) -> list:
    ...

@registry.register(name="json")
def json_loader(path: str) -> dict:
    ...

registry.list()          # ["csv_loader", "json"]
registry.get("json")     # → json_loader
```

## Singleton — une seule instance par classe
```python
from functools import wraps

def singleton(cls):
    instances = {}
    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class AppConfig:
    def __init__(self, env: str = "dev") -> None:
        self.env = env

cfg1 = AppConfig("prod")
cfg2 = AppConfig("dev")    # ignoré — même instance
cfg1 is cfg2               # True ✅
```

## Deprecation warning
```python
import warnings
from functools import wraps

def deprecated(reason: str = ""):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            warnings.warn(
                f"{func.__name__} est déprécié. {reason}",
                DeprecationWarning,
                stacklevel=2,
            )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@deprecated("Utiliser new_function() à la place.")
def old_function(): ...
```

## Context manager depuis un décorateur
```python
from contextlib import contextmanager

@contextmanager
def managed_resource(name: str):
    print(f"Acquisition : {name}")
    resource = {"name": name, "active": True}
    try:
        yield resource
    except Exception as e:
        print(f"Erreur dans {name} : {e}")
        raise
    finally:
        resource["active"] = False
        print(f"Libération : {name}")

with managed_resource("DB Connection") as conn:
    print(f"Utilisation de {conn['name']}")
```
