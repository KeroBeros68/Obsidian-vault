#python #typing #optional #union #none

## None — une fonction sans retour

python

```python
def log(message: str) -> None:
    print(message)
    # pas de return, ou return implicite
```

## Optional — valeur ou None

python

```python
from typing import Optional

# Optional[X] est exactement Union[X, None]
def find_user(name: str) -> Optional[str]:
    if name == "Alice":
        return "alice@example.com"
    return None

# Python 3.10+ — syntaxe courte
def find_user(name: str) -> str | None:
    ...
```

## Union — plusieurs types possibles

python

```python
from typing import Union

def stringify(value: Union[int, float, str]) -> str:
    return str(value)

# Python 3.10+ — syntaxe courte avec |
def stringify(value: int | float | str) -> str:
    return str(value)
```

## Tableau — syntaxes équivalentes

|Ancien (3.5+)|Nouveau (3.10+)|Signification|
|---|---|---|
|`Optional[str]`|`str \| None`|str ou None|
|`Union[str, int]`|`str \| int`|str ou int|
|`Union[str, int, None]`|`str \| int \| None`|str, int, ou None|

## Tester le type au runtime — isinstance

python

```python
def process(value: int | str) -> str:
    if isinstance(value, int):
        return f"Entier : {value}"
    return f"Chaîne : {value}"
    # mypy sait que dans chaque branche, value a le bon type
    # → narrowing (rétrécissement de type)
```

## Narrowing — rétrécissement de type

python

```python
def greet(name: str | None) -> str:
    if name is None:
        return "Anonyme"
    # Ici mypy sait que name est str (pas None)
    return name.upper()   # ✅ pas d'erreur mypy

# Autres formes de narrowing
if isinstance(x, int): ...         # isinstance
if x is not None: ...              # vérification None
assert x is not None               # assertion
if type(x) is str: ...             # type exact
```

> [!tip] Optional[X] n'implique pas que la valeur est "optionnelle" `Optional[str]` signifie que la valeur peut être `None` — pas qu'on peut ne pas passer l'argument. Pour rendre un paramètre optionnel : lui donner une valeur par défaut.
> 
> python
> 
> ```python
> def greet(name: str | None = None) -> str: ...
> # → paramètre optionnel ET peut valoir None
> ```
