#python #typing #fonctions #return #overload

## Syntaxe complète


```python
def create_user(
    name:     str,
    age:      int,
    email:    str | None = None,
    *args:    str,           # *args → tuple[str, ...]
    verbose:  bool = False,  # keyword-only
    **kwargs: int,           # **kwargs → dict[str, int]
) -> dict[str, str]:
    ...
```

## Retours complexes


```python
from typing import Optional

# Retourner un tuple typé
def minmax(values: list[int]) -> tuple[int, int]:
    return min(values), max(values)

low, high = minmax([3, 1, 4, 1, 5])

# Retourner une union
def parse(text: str) -> int | None:
    try:
        return int(text)
    except ValueError:
        return None
```

## NoReturn — fonction qui ne revient jamais


```python
from typing import NoReturn

def crash(message: str) -> NoReturn:
    raise RuntimeError(message)
    # mypy sait que rien après cet appel n'est accessible

def exit_app() -> NoReturn:
    import sys
    sys.exit(1)
```

## @overload — signatures multiples


```python
from typing import overload

@overload
def process(value: int) -> int: ...
@overload
def process(value: str) -> str: ...

def process(value: int | str) -> int | str:
    if isinstance(value, int):
        return value * 2
    return value.upper()

result_int: int = process(5)      # ✅ mypy sait que c'est un int
result_str: str = process("hi")   # ✅ mypy sait que c'est un str
```

## Self — retourner l'instance courante (3.11+)


```python
from typing import Self

class Builder:
    def set_name(self, name: str) -> Self:   # retourne toujours l'instance courante
        self.name = name
        return self

    def set_age(self, age: int) -> Self:
        self.age = age
        return self

# Chaînage de méthodes — mypy suit le type exact même avec l'héritage
class SpecialBuilder(Builder):
    pass

b = SpecialBuilder().set_name("Alice").set_age(30)
# b est SpecialBuilder, pas Builder — Self le garantit
```

## ParamSpec — annoter les décorateurs (3.10+)


```python
from typing import ParamSpec, TypeVar, Callable
from functools import wraps

P = ParamSpec("P")
T = TypeVar("T")

def logged(func: Callable[P, T]) -> Callable[P, T]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        print(f"Appel : {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@logged
def add(a: int, b: int) -> int:
    return a + b

add(1, 2)      # ✅ mypy sait que add prend deux int
add("x", "y") # ❌ mypy détecte l'erreur
```

> [!tip] @overload — les stubs ne s'exécutent pas Les versions décorées `@overload` sont des stubs pour le type checker uniquement. Seule l'implémentation finale (sans `@overload`) s'exécute réellement.
