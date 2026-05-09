#python #typing #pièges #erreurs #debugging

## 🪤 Piège 1 — Les annotations n'ont aucun effet à l'exécution

python

```python
def add(a: int, b: int) -> int:
    return a + b

add("hello", "world")   # ✅ Python l'accepte au runtime !
# ❌ Seul mypy/pyright détecte l'erreur — pas Python lui-même
```

## 🪤 Piège 2 — Optional[str] ≠ paramètre optionnel

python

```python
def greet(name: Optional[str]) -> str:   # ❌ name est OBLIGATOIRE, mais peut valoir None
    ...

def greet(name: Optional[str] = None) -> str:   # ✅ optionnel ET peut valoir None
    ...

def greet(name: str = "Anonyme") -> str:         # ✅ optionnel, ne peut pas valoir None
    ...
```

## 🪤 Piège 3 — List vs list en Python 3.8

python

```python
# Python 3.8 — List doit venir de typing
from typing import List
tags: List[str] = []   # ✅

tags: list[str] = []   # ❌ NameError en Python 3.8

# Python 3.9+ — list[str] nativement
tags: list[str] = []   # ✅
```

## 🪤 Piège 4 — TypeVar mal nommé ou déclaré localement

python

```python
def first(items):
    T = TypeVar("T")   # ❌ déclaré dans la fonction — erreur mypy subtile
    def first(items: list[T]) -> T: ...

T = TypeVar("T")       # ✅ au niveau module
def first(items: list[T]) -> T:
    return items[0]
```

## 🪤 Piège 5 — Oublier return None dans les branches

python

```python
def find(items: list[str], target: str) -> str | None:
    for item in items:
        if item == target:
            return item
    # ❌ mypy : Missing return statement — retourne None implicitement
    # ✅ Ajouter : return None

def find(items: list[str], target: str) -> str | None:
    for item in items:
        if item == target:
            return item
    return None   # ✅ explicite
```

## 🪤 Piège 6 — Muter une Sequence reçue en paramètre

python

```python
from collections.abc import Sequence

def pop_first(items: Sequence[int]) -> int:
    return items.pop(0)   # ❌ Sequence n'a pas .pop() !

# ✅ Utiliser MutableSequence ou list si la mutation est nécessaire
from collections.abc import MutableSequence

def pop_first(items: MutableSequence[int]) -> int:
    return items.pop(0)   # ✅
```

## 🪤 Piège 7 — cast sans vérification runtime

python

```python
from typing import cast

def get_value() -> object:
    return 42

x: str = cast(str, get_value())   # ❌ cast dit "c'est un str" mais c'est un int !
x.upper()                          # ❌ AttributeError à l'exécution

# cast ne fait RIEN au runtime — juste une indication au type checker
```

## 🪤 Piège 8 — TypedDict et clés manquantes

python

```python
from typing import TypedDict

class User(TypedDict):
    name: str
    age:  int

u: User = {"name": "Alice"}   # ❌ mypy : "age" manquant
u: User = {"name": "Alice", "age": 30, "email": "x"}  # ❌ mypy : "email" inconnu
```

## 🪤 Piège 9 — Confondre ClassVar et instance var

python

```python
class Counter:
    count: int = 0     # ← ClassVar ou instance var ? Ambigu !

class Counter:
    count: ClassVar[int] = 0   # ✅ explicite : partagé entre instances
    total: int = 0             # ✅ explicite : propre à chaque instance
```

## 🪤 Piège 10 — Protocol et @runtime_checkable incomplet

python

```python
@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...
    size: int

class Circle:
    def draw(self) -> None: print("O")
    # size absent !

isinstance(Circle(), Drawable)   # ✅ True au runtime (vérifie seulement les méthodes)
# ❌ Mais mypy détecte que size est absent → erreur statique seulement
```

## 🪤 Piège 11 — from **future** import annotations casse get_type_hints

python

```python
from __future__ import annotations

class Config:
    host: str

# Sans le bon namespace, get_type_hints peut échouer
import typing
typing.get_type_hints(Config)               # ❌ peut NameError si types pas importés
typing.get_type_hints(Config, globalns=globals())   # ✅ fournir le contexte
```

## 🪤 Piège 12 — Any propagé silencieusement

python

```python
from typing import Any

def get_data() -> Any:
    return {"name": "Alice", "age": 30}

data = get_data()
name = data["name"]   # name a le type Any → pas de vérification possible
name.nonexistent()    # ❌ bug runtime — mypy n'a pas détecté

# ✅ Caster ou valider dès la sortie de la fonction Any
from pydantic import BaseModel

class Response(BaseModel):
    name: str
    age:  int

response = Response.model_validate(get_data())   # ✅ validé et typé
```

## Récapitulatif rapide

|Piège|Solution|
|---|---|
|Annotations sans effet runtime|Utiliser Pydantic ou beartype pour la validation runtime|
|`Optional[str]` ≠ paramètre optionnel|Ajouter `= None` pour rendre optionnel|
|`list[str]` en Python 3.8|`from typing import List` ou passer à 3.9+|
|TypeVar local|Déclarer au niveau module|
|Return manquant|Toujours retourner explicitement dans toutes les branches|
|Muter une Sequence|Utiliser `MutableSequence` ou `list`|
|cast sans vérification|Réserver aux cas où on est sûr à 100%|
|TypedDict clés manquantes|Toutes les clés sont obligatoires sauf `NotRequired`|
|ClassVar ambigu|Toujours annoter `ClassVar[T]` explicitement|
|Any propagé|Valider / caster dès la sortie de la zone Any|
