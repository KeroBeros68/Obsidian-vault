#python #decorateurs #stdlib #functools #lru_cache

## functools.wraps
```python
from functools import wraps
# Voir [[Déco 03 — Décorateur avec @wraps & préservation des métadonnées]]
```

## functools.lru_cache — mémoïsation automatique
```python
from functools import lru_cache

@lru_cache(maxsize=128)    # cache LRU — 128 entrées max
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(50)              # instantané grâce au cache
fibonacci.cache_info()     # CacheInfo(hits=48, misses=51, maxsize=128, currsize=51)
fibonacci.cache_clear()    # vider le cache

# maxsize=None → cache illimité (équivalent à @cache)
@lru_cache(maxsize=None)
def fib(n): ...
```

## functools.cache — lru_cache sans limite (3.9+)
```python
from functools import cache

@cache                     # cache illimité, plus simple que lru_cache(maxsize=None)
def expensive(n: int) -> int:
    return sum(range(n))
```

## functools.cached_property — propriété mémoïsée
```python
from functools import cached_property

class Circle:
    def __init__(self, radius: float) -> None:
        self.radius = radius

    @cached_property
    def area(self) -> float:
        print("Calcul de l'aire...")
        return 3.14159 * self.radius ** 2
    # Calculé une seule fois, puis stocké dans self.__dict__["area"]

c = Circle(5)
c.area   # "Calcul de l'aire..." puis 78.53...
c.area   # 78.53... — pas de recalcul ✅
```

## functools.partial — application partielle
```python
from functools import partial

def power(base: int, exponent: int) -> int:
    return base ** exponent

square = partial(power, exponent=2)   # fixe exponent=2
cube   = partial(power, exponent=3)

square(5)   # 25
cube(3)     # 27
```

## functools.total_ordering
```python
from functools import total_ordering

@total_ordering                       # génère les méthodes de comparaison manquantes
class Version:
    def __init__(self, major: int, minor: int) -> None:
        self.major = major
        self.minor = minor

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor) == (other.major, other.minor)

    def __lt__(self, other: "Version") -> bool:
        return (self.major, self.minor) < (other.major, other.minor)

# @total_ordering génère automatiquement __le__, __gt__, __ge__
v1 = Version(1, 2)
v2 = Version(1, 3)
v1 < v2    # ✅ True
v1 > v2    # ✅ False — généré par @total_ordering
v1 <= v2   # ✅ True  — généré par @total_ordering
```

## contextlib.contextmanager — créer un context manager
```python
from contextlib import contextmanager
import time

@contextmanager
def timer(label: str):
    start = time.perf_counter()
    try:
        yield                          # exécution du bloc with
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label} : {elapsed:.4f}s")

with timer("Ma boucle"):
    total = sum(range(10_000_000))
# Ma boucle : 0.3821s
```

## dataclasses.dataclass
```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    label: str = ""

@dataclass(frozen=True)   # immuable + hashable
class Config:
    host: str
    port: int = 5432

@dataclass(order=True)    # génère __lt__, __le__, __gt__, __ge__
class Version:
    major: int
    minor: int
```
