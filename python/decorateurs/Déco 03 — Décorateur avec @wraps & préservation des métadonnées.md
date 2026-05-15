#python #decorateurs #wraps #functools #métadonnées

## Le problème sans @wraps
```python
def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@decorator
def add(a: int, b: int) -> int:
    """Additionne deux entiers."""
    return a + b

print(add.__name__)       # "wrapper"  ❌
print(add.__doc__)        # None       ❌
print(add.__annotations__)# {}         ❌
help(add)                 # montre la doc de wrapper, pas add ❌
```

## La solution — @wraps
```python
from functools import wraps

def decorator(func):
    @wraps(func)              # copie __name__, __doc__, __annotations__,
    def wrapper(*args, **kwargs):  # __module__, __qualname__, __dict__, __wrapped__
        return func(*args, **kwargs)
    return wrapper

@decorator
def add(a: int, b: int) -> int:
    """Additionne deux entiers."""
    return a + b

print(add.__name__)        # "add"                         ✅
print(add.__doc__)         # "Additionne deux entiers."     ✅
print(add.__annotations__) # {"a": int, "b": int, "return": int} ✅
add.__wrapped__            # la fonction originale non décorée ✅
```

## __wrapped__ — accéder à la fonction originale
```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("décorée")
        return func(*args, **kwargs)
    return wrapper

@decorator
def greet(name: str) -> str:
    return f"Bonjour {name}"

greet("Alice")              # "décorée" puis "Bonjour Alice"
greet.__wrapped__("Alice")  # "Bonjour Alice" — sans le décorateur
```

## update_wrapper — équivalent manuel de @wraps
```python
from functools import update_wrapper, WRAPPER_ASSIGNMENTS

def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    update_wrapper(wrapper, func)   # équivalent à @wraps(func)
    return wrapper

# WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__',
#                        '__annotations__', '__doc__')
# WRAPPER_UPDATES     = ('__dict__',)
```

## Introspection après décoration
```python
import inspect

@decorator
def add(a: int, b: int) -> int:
    """Additionne."""
    return a + b

inspect.signature(add)      # (a: int, b: int) -> int  ✅
inspect.getdoc(add)         # "Additionne."            ✅
inspect.unwrap(add)         # fonction originale        ✅ (suit __wrapped__)
```

> [!warning] Certains frameworks lisent __name__ et __doc__
> FastAPI lit `__name__` pour nommer les routes, Pydantic lit `__annotations__`.
> Sans `@wraps`, les endpoints FastAPI peuvent se nommer tous "wrapper" et bugger à la registration.
