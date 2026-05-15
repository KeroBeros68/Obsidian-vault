#python #decorateurs #classe #__call__ #état

## Pourquoi une classe ?
Une classe est plus lisible qu'une closure à trois niveaux quand :
- L'état est complexe (plusieurs attributs)
- On veut des méthodes auxiliaires (`.reset()`, `.stats()`)
- On veut que l'objet soit inspectable facilement

## Décorateur classe — syntaxe de base
```python
from functools import update_wrapper
from typing import Callable, Any

class MyDecorator:
    def __init__(self, func: Callable) -> None:
        update_wrapper(self, func)   # copie __name__, __doc__, etc. sur self
        self.func = func

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        print("Avant")
        result = self.func(*args, **kwargs)
        print("Après")
        return result

@MyDecorator
def greet(name: str) -> str:
    return f"Bonjour {name}"

greet("Alice")    # Avant → Bonjour Alice → Après
type(greet)       # <class 'MyDecorator'>
```

## Compteur d'appels — classe vs closure
```python
class CountCalls:
    def __init__(self, func: Callable) -> None:
        update_wrapper(self, func)
        self.func       = func
        self.call_count = 0

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        self.call_count += 1
        return self.func(*args, **kwargs)

    def reset(self) -> None:
        self.call_count = 0

    def stats(self) -> dict:
        return {"function": self.func.__name__, "calls": self.call_count}

@CountCalls
def add(a: int, b: int) -> int:
    return a + b

add(1, 2)
add(3, 4)
add.call_count   # 2
add.stats()      # {"function": "add", "calls": 2}
add.reset()
add.call_count   # 0
```

## Décorateur classe avec arguments
```python
from functools import wraps

class Retry:
    def __init__(self, max_attempts: int = 3, exceptions: tuple = (Exception,)) -> None:
        self.max_attempts = max_attempts
        self.exceptions   = exceptions

    def __call__(self, func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            for attempt in range(1, self.max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except self.exceptions as e:
                    if attempt == self.max_attempts:
                        raise
                    print(f"Tentative {attempt} échouée : {e}")
        return wrapper

@Retry(max_attempts=3, exceptions=(ConnectionError,))
def fetch(url: str) -> dict:
    ...
```

## Décorateur classe sur une méthode — problème de descriptor
```python
# ❌ Un décorateur classe ne fonctionne pas directement sur une méthode
class MyDeco:
    def __init__(self, func): self.func = func
    def __call__(self, *a, **kw): return self.func(*a, **kw)

class MyClass:
    @MyDeco
    def method(self): ...    # ❌ self n'est pas passé correctement

# ✅ Solution : implémenter __get__ pour supporter le protocole descriptor
class MyDeco:
    def __init__(self, func): self.func = func

    def __call__(self, *args, **kwargs):
        return self.func(*args, **kwargs)

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        from functools import partial
        return partial(self, obj)       # bind self à la méthode
```

> [!tip] Classe vs closure — quand choisir ?
> Fermeture (closure) → décorateur simple, peu d'état, code compact
> Classe → état riche, méthodes `.reset()` / `.stats()`, lisibilité importante
