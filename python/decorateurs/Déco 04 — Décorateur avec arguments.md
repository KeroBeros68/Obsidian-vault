#python #decorateurs #arguments #factory

## Le problème — un décorateur ne peut pas prendre d'arguments directement
```python
# ❌ Ceci ne fonctionne pas
@repeat(3)
def hello(): ...
# Python évalue @repeat(3) AVANT de l'appliquer
# Il faut que repeat(3) retourne un décorateur
```

## Solution — fabrique de décorateur (decorator factory)
```python
from functools import wraps

def repeat(times: int):                    # 1. fabrique : prend les arguments
    def decorator(func):                   # 2. décorateur : prend la fonction
        @wraps(func)
        def wrapper(*args, **kwargs):      # 3. wrapper : exécuté à l'appel
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def hello(name: str) -> None:
    print(f"Bonjour {name}")

hello("Alice")
# Bonjour Alice
# Bonjour Alice
# Bonjour Alice
```

## Anatomie d'un décorateur avec arguments
```
repeat(3)         → appel de la fabrique → retourne decorator
@decorator        → appliqué à hello    → retourne wrapper
hello("Alice")    → appel de wrapper    → appelle func 3 fois
```

## Décorateur avec arguments optionnels — @deco ou @deco()
```python
from functools import wraps

def log(func=None, *, level: str = "INFO"):
    """Peut s'utiliser comme @log ou @log(level="DEBUG")."""
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            print(f"[{level}] Appel : {f.__name__}")
            return f(*args, **kwargs)
        return wrapper

    if func is not None:
        # Appelé comme @log — func est la fonction décorée
        return decorator(func)
    # Appelé comme @log(level="DEBUG") — retourner le décorateur
    return decorator

@log                         # ✅ sans parenthèses
def add(a, b): return a + b

@log(level="DEBUG")          # ✅ avec arguments
def multiply(a, b): return a * b
```

## Décorateur avec arguments — exemple pratique : retry
```python
from functools import wraps
import time

def retry(max_attempts: int = 3, delay: float = 1.0, exceptions: tuple = (Exception,)):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exc = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exc = e
                    print(f"Tentative {attempt}/{max_attempts} échouée : {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise last_exc
        return wrapper
    return decorator

@retry(max_attempts=3, delay=0.5, exceptions=(ConnectionError, TimeoutError))
def fetch_data(url: str) -> dict:
    ...   # peut lever ConnectionError
```

> [!tip] Triple niveau d'imbrication
> `fabrique(args)` → `decorator(func)` → `wrapper(*args, **kwargs)`
> C'est le pattern standard. Chaque niveau a un rôle précis : capturer les args, capturer la fonction, exécuter.
