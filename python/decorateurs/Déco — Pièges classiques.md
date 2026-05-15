#python #decorateurs #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier @wraps → perte des métadonnées
```python
def decorator(func):
    def wrapper(*args, **kwargs):   # ❌ pas de @wraps
        return func(*args, **kwargs)
    return wrapper

@decorator
def add(a: int, b: int) -> int:
    """Additionne deux entiers."""
    return a + b

add.__name__   # "wrapper" ❌ — cassé pour FastAPI, les tests, help()
add.__doc__    # None      ❌

# ✅ Toujours :
from functools import wraps
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

## 🪤 Piège 2 — Décorateur appelé au lieu d'appliqué
```python
@my_decorator()    # ❌ si my_decorator ne prend pas d'arguments
def hello(): ...
# my_decorator() est appelé avec 0 args → retourne None → None(hello) → TypeError

@my_decorator      # ✅ sans parenthèses pour un décorateur sans arguments
def hello(): ...
```

## 🪤 Piège 3 — Oublier de retourner la valeur dans le wrapper
```python
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)   # ❌ résultat ignoré !
    return wrapper

@decorator
def add(a, b): return a + b

add(1, 2)   # None ❌ — le résultat est perdu

# ✅
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)   # toujours retourner
```

## 🪤 Piège 4 — Décorateur exécuté à la définition, pas à l'appel
```python
def register(func):
    print(f"Enregistrement de {func.__name__}")   # ← s'exécute à l'import !
    return func

@register
def my_func(): ...
# "Enregistrement de my_func" s'affiche à l'import du module, pas à l'appel
```

## 🪤 Piège 5 — @lru_cache sur une méthode d'instance
```python
from functools import lru_cache

class MyClass:
    @lru_cache(maxsize=128)
    def compute(self, n: int) -> int:   # ❌ self est hashé → fuite mémoire !
        return n * 2                     # chaque instance est mise en cache

# ✅ Solution 1 : @cached_property pour les propriétés
# ✅ Solution 2 : @lru_cache seulement sur les méthodes statiques/fonctions pures
# ✅ Solution 3 : utiliser methodtools ou cachetools
```

## 🪤 Piège 6 — Ordre des décorateurs @classmethod et @staticmethod
```python
class MyClass:
    @my_decorator
    @classmethod              # ❌ my_decorator reçoit un classmethod descriptor
    def method(cls): ...

    @classmethod              # ✅ @classmethod en dernier (le plus haut)
    @my_decorator
    def method(cls): ...
```

## 🪤 Piège 7 — Closure capturant une variable mutable
```python
decorators = []
for i in range(3):
    def deco(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"Step {i}")   # ❌ i vaut 2 pour tous !
            return func(*args, **kwargs)
        return wrapper
    decorators.append(deco)

# ✅ Capturer i par valeur
for i in range(3):
    def deco(func, step=i):   # step=i capture la valeur courante
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"Step {step}")
            return func(*args, **kwargs)
        return wrapper
    decorators.append(deco)
```

## 🪤 Piège 8 — Décorateur de classe casse isinstance
```python
@singleton
class MyClass: ...

isinstance(MyClass(), MyClass)   # ❌ False — MyClass est maintenant la fonction get_instance

# ✅ Solution : vérifier le type directement
type(MyClass()) is MyClass.__wrapped__
```

## 🪤 Piège 9 — État partagé entre toutes les instances décorées
```python
cache = {}   # ❌ au niveau module → partagé entre TOUTES les fonctions décorées

def add_cache(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        if args not in cache:   # partage involontaire entre fonctions
            cache[args] = func(*args, **kwargs)
        return cache[args]
    return wrapper

# ✅ Déclarer le cache dans la closure
def add_cache(func):
    cache = {}                  # un dict par fonction décorée
    @wraps(func)
    def wrapper(*args, **kwargs):
        if args not in cache:
            cache[args] = func(*args, **kwargs)
        return cache[args]
    return wrapper
```

## 🪤 Piège 10 — Décorateur asynchrone sur une fonction synchrone
```python
def async_decorator(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):   # ❌ retourne une coroutine même si func est sync
        return func(*args, **kwargs)
    return wrapper

@async_decorator
def sync_function(): return 42

sync_function()   # → retourne une coroutine, pas 42 ❌

# ✅ Vérifier si la fonction est async
import asyncio, inspect

def smart_decorator(func):
    if inspect.iscoroutinefunction(func):
        @wraps(func)
        async def async_wrapper(*args, **kwargs):
            return await func(*args, **kwargs)
        return async_wrapper
    else:
        @wraps(func)
        def sync_wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return sync_wrapper
```

## 🪤 Piège 11 — @cached_property et frozen dataclass
```python
from dataclasses import dataclass
from functools import cached_property

@dataclass(frozen=True)
class Circle:
    radius: float

    @cached_property              # ❌ frozen=True interdit l'écriture dans __dict__
    def area(self) -> float:
        return 3.14 * self.radius ** 2

# ✅ Utiliser @property simple (sans cache) avec frozen=True
# ✅ Ou ne pas utiliser frozen=True si un cache est nécessaire
```

## 🪤 Piège 12 — Perte de la signature avec *args/**kwargs non typés
```python
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):   # ❌ signature perdue pour le type checker
        return func(*args, **kwargs)
    return wrapper

# ✅ Utiliser ParamSpec pour préserver la signature
from typing import ParamSpec, TypeVar
from collections.abc import Callable

P = ParamSpec("P")
R = TypeVar("R")

def decorator(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        return func(*args, **kwargs)
    return wrapper
```

## Récapitulatif rapide
| Piège | Solution |
|---|---|
| Oublier `@wraps` | Toujours `@wraps(func)` dans le wrapper |
| Décorateur appelé avec `()` inutiles | `@deco` sans parenthèses si pas d'arguments |
| Oublier `return` dans le wrapper | `return func(*args, **kwargs)` systématiquement |
| `@classmethod` dans le mauvais ordre | `@classmethod` en dernier (le plus haut) |
| `@lru_cache` sur méthode d'instance | Utiliser `@cached_property` ou des fonctions pures |
| Variable capturée par référence | Capturer par valeur avec `arg=val` |
| Décorateur async sur fonction sync | Vérifier `inspect.iscoroutinefunction` |
| `@cached_property` avec `frozen=True` | Incompatibles — choisir l'un ou l'autre |
| État au niveau module | Déclarer le cache/état dans la closure |
