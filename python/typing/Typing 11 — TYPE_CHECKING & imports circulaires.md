#python #typing #type_checking #forward-reference #imports

## Le problème — import circulaire


```python
# users.py
from orders import Order   # ← importe orders

# orders.py
from users import User     # ← importe users → import circulaire !
```

## Solution — TYPE_CHECKING


```python
# users.py
from __future__ import annotations   # évalue les annotations en lazy
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from orders import Order   # importé SEULEMENT par le type checker, pas à l'exécution

class User:
    def get_orders(self) -> list["Order"]:   # guillemets si pas de __future__
        ...
```

## from **future** import annotations


```python
# PEP 563 — toutes les annotations sont des chaînes évaluées lazily
from __future__ import annotations

class Node:
    def __init__(self, value: int, next: Node | None = None) -> None:
        # Sans __future__, "Node" n'existe pas encore à cette ligne → NameError
        self.value = value
        self.next  = next
```

## Forward reference — guillemets


```python
# Sans __future__ : citer le type en string pour les références en avant
class Tree:
    left:  "Tree | None" = None   # guillemets car Tree n'est pas encore défini
    right: "Tree | None" = None
    value: int = 0

# Avec __future__ import annotations : guillemets inutiles
from __future__ import annotations

class Tree:
    left:  Tree | None = None   # ✅ pas besoin de guillemets
    right: Tree | None = None
```

## TYPE_CHECKING — pattern complet


```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    # Imports lourds ou circulaires — présents uniquement pour le type checker
    from mymodule import HeavyClass
    from collections.abc import Sequence

class MyService:
    def process(self, items: Sequence[HeavyClass]) -> None:
        ...   # HeavyClass n'est pas importé à l'exécution → pas de coût
```

## typing.get_type_hints() — résoudre les forward refs


```python
import typing

class Config:
    host: str
    port: int

# get_type_hints résout les strings en vrais types
hints = typing.get_type_hints(Config)
# {"host": <class 'str'>, "port": <class 'int'>}
# Pydantic utilise cette fonction en interne
```

> [!warning] from **future** import annotations change le comportement runtime Avec `from __future__ import annotations`, `__annotations__` contient des **strings**, pas des types. Les frameworks qui lisent `__annotations__` directement (certaines librairies anciennes) peuvent bugger. Pydantic v2 gère correctement les deux modes.
