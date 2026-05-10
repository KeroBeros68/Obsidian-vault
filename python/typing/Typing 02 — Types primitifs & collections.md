#python #typing #collections #list #dict #tuple #set

## Types primitifs


```python
name:   str   = "Alice"
age:    int   = 30
score:  float = 9.5
active: bool  = True
data:   bytes = b"hello"
```

## Listes


```python
# Python 3.9+
tags:   list[str]   = ["python", "typing"]
scores: list[int]   = [10, 20, 30]
matrix: list[list[float]] = [[1.0, 2.0], [3.0, 4.0]]

# Python 3.5–3.8 (typing module)
from typing import List
tags: List[str] = ["python"]
```

## Dictionnaires


```python
# Python 3.9+
ages:    dict[str, int]         = {"Alice": 30, "Bob": 25}
config:  dict[str, list[str]]   = {"hosts": ["a", "b"]}
nested:  dict[str, dict[str, int]] = {}

# Python 3.5–3.8
from typing import Dict
ages: Dict[str, int] = {}
```

## Tuples


```python
# Tuple de longueur fixe — chaque position a son type
point:    tuple[int, int]          = (10, 20)
rgb:      tuple[int, int, int]     = (255, 128, 0)
person:   tuple[str, int, float]   = ("Alice", 30, 9.5)

# Tuple de longueur variable — un seul type, ... (ellipsis)
coords:   tuple[float, ...]        = (1.0, 2.0, 3.0, 4.0)

# Python 3.5–3.8
from typing import Tuple
point: Tuple[int, int] = (10, 20)
```

## Sets & frozensets


```python
unique_tags: set[str]        = {"python", "typing"}
immutable:   frozenset[int]  = frozenset({1, 2, 3})
```

## Séquences et mappings abstraits


```python
from collections.abc import Sequence, Mapping, MutableMapping

# Sequence = tout ce qui est indexable (list, tuple, str...)
def first(items: Sequence[int]) -> int:
    return items[0]

first([1, 2, 3])   # ✅ list
first((1, 2, 3))   # ✅ tuple

# Mapping = tout ce qui se comporte comme un dict (en lecture)
def get_name(data: Mapping[str, str]) -> str:
    return data["name"]
```

## Type Any — désactiver la vérification


```python
from typing import Any

def process(data: Any) -> Any:
    return data   # aucune vérification — équivalent à pas d'annotation
```

> [!warning] Éviter Any autant que possible `Any` désactive toute vérification de type. À réserver aux cas où le type est vraiment inconnu ou dynamique. Préférer `object` si tu veux juste signifier "n'importe quel objet".

> [!tip] list vs List (Python 3.9+) En Python 3.9+, utiliser `list[str]` directement. En 3.8 et avant, importer `List` depuis `typing`. Les deux coexistent — `List[str]` fonctionne encore en 3.12.
