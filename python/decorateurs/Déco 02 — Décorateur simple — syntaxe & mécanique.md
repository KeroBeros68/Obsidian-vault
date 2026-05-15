#python #decorateurs #syntaxe #wrapper

## Mécanique — sans sucre syntaxique
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Avant l'appel")
        result = func(*args, **kwargs)   # appel de la fonction originale
        print("Après l'appel")
        return result
    return wrapper

def greet(name: str) -> str:
    return f"Bonjour {name}"

# Application manuelle
greet = my_decorator(greet)   # greet est maintenant wrapper
greet("Alice")
# Avant l'appel
# Après l'appel
# → "Bonjour Alice"
```

## Syntaxe @ — sucre syntaxique
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Avant l'appel")
        result = func(*args, **kwargs)
        print("Après l'appel")
        return result
    return wrapper

@my_decorator                 # ← équivalent à greet = my_decorator(greet)
def greet(name: str) -> str:
    return f"Bonjour {name}"

greet("Alice")
# Avant l'appel
# Après l'appel
```

## Template universel d'un décorateur
```python
from functools import wraps
from collections.abc import Callable
from typing import Any

def my_decorator(func: Callable) -> Callable:
    @wraps(func)                          # préserve __name__, __doc__, etc.
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        # — code avant —
        result = func(*args, **kwargs)
        # — code après —
        return result
    return wrapper
```

## *args et **kwargs — passer tous les arguments
```python
def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Appel : {func.__name__}({args}, {kwargs})")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def add(a: int, b: int) -> int:
    return a + b

add(1, 2)         # Appel : add((1, 2), {})
add(a=1, b=2)     # Appel : add((), {"a": 1, "b": 2})
add(1, b=2)       # Appel : add((1,), {"b": 2})
```

## Vérifier que le décorateur fonctionne
```python
@my_decorator
def hello(): pass

# Sans @wraps → les métadonnées sont perdues
print(hello.__name__)   # "wrapper" ❌
print(hello.__doc__)    # None ❌

# Avec @wraps → les métadonnées sont préservées
print(hello.__name__)   # "hello" ✅
print(hello.__doc__)    # docstring originale ✅
```

> [!important] Toujours utiliser @wraps(func)
> Sans `@wraps`, la fonction décorée perd son nom, sa docstring, ses annotations.
> Cela casse `help()`, les outils de debugging, Pydantic, FastAPI, et les tests.
> Voir [[Déco 03 — Décorateur avec @wraps & préservation des métadonnées]].
