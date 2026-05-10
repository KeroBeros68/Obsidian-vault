#python #typing #protocol #duck-typing #structural

## Le problème — isinstance est trop rigide


```python
# Pour annoter "tout objet avec une méthode .draw()"
# sans héritage forcé → Protocol

from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("O")

class Square:
    def draw(self) -> None:
        print("[]")

# Circle et Square n'héritent PAS de Drawable
# Mais ils satisfont le protocole structurellement

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())   # ✅
render(Square())   # ✅
render("string")   # ❌ mypy : str n'a pas de méthode draw()
```

## Protocol avec attributs


```python
from typing import Protocol

class HasName(Protocol):
    name: str              # attribut requis
    def greet(self) -> str: ...   # méthode requise

class User:
    def __init__(self, name: str):
        self.name = name

    def greet(self) -> str:
        return f"Bonjour {self.name}"

def introduce(entity: HasName) -> str:
    return entity.greet()

introduce(User("Alice"))   # ✅
```

## @runtime_checkable — isinstance avec Protocol


```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Sizeable(Protocol):
    def __len__(self) -> int: ...

isinstance([1, 2, 3], Sizeable)   # ✅ True
isinstance("hello", Sizeable)     # ✅ True
isinstance(42, Sizeable)          # ❌ False
```

> [!warning] @runtime_checkable ne vérifie que la présence des méthodes Il vérifie que les méthodes existent, pas leurs signatures. `isinstance(obj, Sizeable)` vérifie juste que `obj` a un `__len__`, pas son type de retour.

## Protocoles standards de Python (collections.abc)


```python
from collections.abc import (
    Iterable,    # __iter__
    Iterator,    # __iter__ + __next__
    Sequence,    # __getitem__ + __len__
    Mapping,     # __getitem__ + __len__ + __iter__
    Callable,    # __call__
    Hashable,    # __hash__
    Sized,       # __len__
    Container,   # __contains__
)

def process_all(items: Iterable[int]) -> list[int]:
    return [x * 2 for x in items]

process_all([1, 2, 3])        # ✅ list
process_all((1, 2, 3))        # ✅ tuple
process_all(range(10))        # ✅ range
process_all({1, 2, 3})        # ✅ set
```

> [!tip] Protocol vs ABC `Protocol` = **duck typing structurel** — pas besoin d'hériter, mypy vérifie la structure. `ABC` = **héritage nominal** — la classe doit hériter explicitement. Préférer `Protocol` pour les interfaces légères et les bibliothèques tierces.
