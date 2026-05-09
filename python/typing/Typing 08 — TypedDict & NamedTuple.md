#python #typing #typeddict #namedtuple #dataclass

## TypedDict — dict avec types de clés fixes

python

```python
from typing import TypedDict

class UserDict(TypedDict):
    name:  str
    age:   int
    email: str

user: UserDict = {"name": "Alice", "age": 30, "email": "alice@x.com"}
user["name"]   # ✅ mypy sait que c'est un str
user["phone"]  # ❌ mypy : TypedDict has no key "phone"

# Clés optionnelles
from typing import NotRequired   # Python 3.11+

class Config(TypedDict):
    host:  str
    port:  int
    debug: NotRequired[bool]   # clé optionnelle

cfg: Config = {"host": "localhost", "port": 5432}   # ✅ debug absent
```

## TypedDict — syntaxe alternative (pour clés invalides en Python)

python

```python
# Si la clé n'est pas un identifiant Python valide
WeirdDict = TypedDict("WeirdDict", {"content-type": str, "x-auth": str})
```

## TypedDict vs dict simple

||`dict[str, Any]`|`TypedDict`|
|---|---|---|
|Clés vérifiées|❌|✅|
|Types des valeurs|❌|✅|
|Clés obligatoires|❌|✅|
|Héritage|❌|✅|
|Runtime overhead|aucun|aucun|

## NamedTuple — tuple avec champs nommés et typés

python

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float
    label: str = ""   # valeur par défaut

p = Point(1.0, 2.0)
print(p.x)      # 1.0
print(p[0])     # 1.0 — accès par index toujours possible
p.x = 3.0       # ❌ NamedTuple est immuable

# Déstructuration
x, y, label = p
```

## NamedTuple vs dataclass vs TypedDict

||`NamedTuple`|`dataclass`|`TypedDict`|
|---|---|---|---|
|Immuable|✅ (par défaut)|❌ (sauf frozen)|❌|
|Accès par index|✅|❌|❌|
|C'est un dict|❌|❌|✅|
|Héritage|limité|✅|✅|
|Méthodes custom|✅|✅|❌|

> [!tip] Quand utiliser quoi ? `TypedDict` → data brute JSON/API, compatible avec les dicts existants `NamedTuple` → données légères immuables, remplacement de tuple `dataclass` → objets avec comportement et mutabilité
