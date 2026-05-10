#python #typing #callable #iterator #generator

## Callable — annoter une fonction en paramètre


```python
from collections.abc import Callable

# Callable[[types des args], type de retour]
def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)

def double(x: int) -> int:
    return x * 2

apply(double, 5)             # ✅
apply(lambda x: x + 1, 5)   # ✅

# Callable sans args spécifiés — [...] pour n'importe quelle signature
def run(callback: Callable[..., None]) -> None:
    callback()
```

## Callable avec plusieurs paramètres


```python
# Callable[[arg1, arg2, ...], retour]
Transformer = Callable[[str, int], str]

def repeat(text: str, times: int) -> str:
    return text * times

def apply_transform(func: Transformer, s: str, n: int) -> str:
    return func(s, n)

apply_transform(repeat, "ab", 3)   # "ababab"
```

## Iterator & Iterable


```python
from collections.abc import Iterator, Iterable

def countdown(n: int) -> Iterator[int]:
    while n > 0:
        yield n
        n -= 1

def sum_all(items: Iterable[int]) -> int:
    return sum(items)

sum_all([1, 2, 3])         # ✅
sum_all(countdown(5))      # ✅
sum_all(range(10))         # ✅
```

## Generator — yield complet


```python
from collections.abc import Generator

# Generator[YieldType, SendType, ReturnType]
def integers(start: int = 0) -> Generator[int, None, None]:
    n = start
    while True:
        yield n
        n += 1

# Generator qui reçoit des valeurs via .send()
def accumulator() -> Generator[float, float, str]:
    total = 0.0
    while True:
        value: float = yield total
        total += value
    return "done"   # jamais atteint mais typé
```

## AsyncIterator & AsyncGenerator


```python
from collections.abc import AsyncIterator, AsyncGenerator

async def async_range(n: int) -> AsyncIterator[int]:
    for i in range(n):
        yield i

async def fetch_items() -> AsyncGenerator[str, None]:
    for item in ["a", "b", "c"]:
        yield item
```

> [!tip] Simplifier avec Iterator[T] Pour la plupart des générateurs simples, `Iterator[T]` suffit. `Generator[Y, S, R]` est nécessaire uniquement quand tu utilises `.send()` ou le type de retour.
