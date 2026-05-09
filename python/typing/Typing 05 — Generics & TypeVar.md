#python #typing #generics #typevar #generic

## TypeVar — paramètre de type

python

```python
from typing import TypeVar

T = TypeVar("T")   # T peut être n'importe quel type

def identity(value: T) -> T:
    return value

x: int = identity(42)     # T = int
s: str = identity("hi")   # T = str
# mypy infère T à chaque appel
```

## TypeVar avec contraintes

python

```python
from typing import TypeVar

# T ne peut être que int ou float
Numeric = TypeVar("Numeric", int, float)

def double(value: Numeric) -> Numeric:
    return value * 2

double(5)     # ✅ int
double(3.14)  # ✅ float
double("hi")  # ❌ mypy : Value of type variable "Numeric" cannot be "str"
```

## TypeVar avec bound — sous-type de

python

```python
from typing import TypeVar

class Animal:
    def speak(self) -> str: ...

class Dog(Animal):
    def fetch(self) -> None: ...

A = TypeVar("A", bound=Animal)

def make_speak(animal: A) -> A:
    animal.speak()
    return animal   # retourne le même sous-type qu'en entrée

dog: Dog = make_speak(Dog())   # ✅ retourne Dog, pas juste Animal
```

## Classes génériques (Python 3.9+)

python

```python
from typing import Generic, TypeVar

T = TypeVar("T")

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

    def peek(self) -> T | None:
        return self._items[-1] if self._items else None

stack: Stack[int] = Stack()
stack.push(1)
stack.push(2)
value: int = stack.pop()   # ✅ mypy sait que c'est un int
```

## Plusieurs TypeVars

python

```python
from typing import TypeVar, Generic

K = TypeVar("K")
V = TypeVar("V")

class Pair(Generic[K, V]):
    def __init__(self, key: K, value: V) -> None:
        self.key   = key
        self.value = value

    def swap(self) -> "Pair[V, K]":
        return Pair(self.value, self.key)

p: Pair[str, int] = Pair("age", 30)
swapped: Pair[int, str] = p.swap()
```

## TypeVarTuple — Generics variadiques (3.11+)

python

```python
from typing import TypeVarTuple, Unpack

Ts = TypeVarTuple("Ts")

def zip_values(*args: Unpack[Ts]) -> tuple[Unpack[Ts]]:
    return args

result: tuple[int, str, bool] = zip_values(1, "a", True)
```

> [!tip] Nommage des TypeVar Convention : une seule lettre majuscule (`T`, `K`, `V`) ou un nom suffixé par `T` (`ItemT`, `KeyT`). Déclarer les TypeVar au niveau module, pas dans les fonctions.
