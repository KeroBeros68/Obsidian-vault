#python #typing #annotations #bases

## Pourquoi annoter ?

Les annotations de type sont **optionnelles** en Python (le langage reste dynamique), mais elles apportent :

- Détection d'erreurs **avant l'exécution** (mypy, pyright)
- Autocomplétion et refactoring dans l'IDE
- Documentation vivante du code
- Interopérabilité avec Pydantic, FastAPI, dataclasses...


```python
# Sans annotations — Python accepte, l'IDE ne sait rien
def add(a, b):
    return a + b

# Avec annotations — l'IDE et mypy vérifient les appels
def add(a: int, b: int) -> int:
    return a + b

add(1, 2)      # ✅
add("x", "y") # ❌ mypy : Argument 1 has incompatible type "str"; expected "int"
```

## Syntaxe de base — variables


```python
# Annotation de variable (PEP 526 — Python 3.6+)
name:  str  = "Alice"
age:   int  = 30
score: float = 9.5
active: bool = True

# Annotation sans valeur — déclare le type sans affecter
x: int   # valide Python — x n'existe pas encore
```

## Annotations de fonction


```python
def greet(name: str) -> str:
    return f"Bonjour {name}"

def add(a: int, b: int = 0) -> int:   # paramètre avec défaut
    return a + b

def nothing() -> None:                  # pas de retour
    print("hello")
```

## Les annotations sont des métadonnées


```python
# Inspecter les annotations au runtime
def add(a: int, b: int) -> int: ...

print(add.__annotations__)
# {"a": <class 'int'>, "b": <class 'int'>, "return": <class 'int'>}

import typing
typing.get_type_hints(add)   # résolution des forward refs incluse
```

## Versions Python et module typing

|Version|Nouveauté|
|---|---|
|3.5|`typing` module, `List`, `Dict`, `Optional`|
|3.9|`list[str]` au lieu de `List[str]` directement|
|3.10|`str \| None` au lieu de `Union[str, None]`|
|3.11|`Self`, `Never`, `LiteralString`|
|3.12|`type` alias natif (`type Point = tuple[int, int]`)|

> [!tip] Python 3.9+ — syntaxe allégée En Python 3.9+, plus besoin d'importer `List`, `Dict`, `Tuple` depuis `typing`. Utiliser directement `list[str]`, `dict[str, int]`, `tuple[int, ...]`.

> [!warning] Les annotations ne sont pas vérifiées à l'exécution Python **ignore** les annotations au runtime — elles ne produisent aucune erreur seules. Pour une validation runtime, utiliser **Pydantic** ou **beartype**.
