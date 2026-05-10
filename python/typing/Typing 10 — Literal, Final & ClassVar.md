#python #typing #literal #final #classvar

## Literal — valeurs exactes autorisées


```python
from typing import Literal

# Restreindre à des valeurs fixes
Direction = Literal["north", "south", "east", "west"]
Status    = Literal["pending", "active", "closed"]
Port      = Literal[80, 443, 8080]

def move(direction: Direction) -> None: ...
def set_status(s: Status) -> None: ...

move("north")   # ✅
move("up")      # ❌ mypy : Argument 1 has incompatible type "Literal['up']"

# Combiner des Literal
Mode = Literal["r", "w", "rb", "wb"]
```

## Literal pour le narrowing


```python
def handle(event: Literal["click", "hover", "focus"]) -> str:
    if event == "click":
        return "clicked"   # mypy sait que event == "click" ici
    elif event == "hover":
        return "hovered"
    return "focused"
```

## Final — constante non réassignable


```python
from typing import Final

MAX_SIZE:  Final = 100
API_URL:   Final[str] = "https://api.example.com"
PI:        Final[float] = 3.14159

MAX_SIZE = 200   # ❌ mypy : Cannot assign to final name "MAX_SIZE"

# Final dans une classe
class Config:
    MAX_RETRIES: Final = 3
    BASE_URL:    Final[str] = "https://api.example.com"
```

## ClassVar — variable de classe vs instance


```python
from typing import ClassVar

class Counter:
    count: ClassVar[int] = 0    # partagé entre toutes les instances
    name:  str                   # propre à chaque instance

    def __init__(self, name: str) -> None:
        Counter.count += 1
        self.name = name

c = Counter("Alice")
Counter.count   # ✅ accès via la classe
c.count         # ✅ accès via l'instance (lit la ClassVar)
c.count = 5     # ❌ mypy : Cannot assign to class variable "count" via instance
```

## Literal vs Enum — quand choisir ?

||`Literal`|`Enum`|
|---|---|---|
|Déclaration|inline|classe dédiée|
|Itération|❌|✅|
|Valeurs dynamiques|❌|✅|
|Lisibilité|✅ pour peu de valeurs|✅ pour beaucoup|
|Sérialisation JSON|directe|`.value` requis|
|Pydantic|✅|✅|

> [!tip] Final pour les constantes de module Déclarer les constantes de module avec `Final` permet à mypy de détecter les réassignations accidentelles. `MAX_SIZE: Final = 100` est plus sûr que `MAX_SIZE = 100`.
