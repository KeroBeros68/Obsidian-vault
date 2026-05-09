#pydantic #config #configdict #comportement

## Syntaxe

```python
from pydantic import BaseModel, ConfigDict

class MyModel(BaseModel):
    model_config = ConfigDict(strict=True, frozen=True, extra="forbid")
    name: str
    age:  int
```

## Toutes les options utiles

```python
model_config = ConfigDict(
    strict               = True,             # désactive la coercition
    arbitrary_types_allowed = True,          # autorise les types non-Pydantic
    frozen               = True,             # immuable + hashable
    extra                = "forbid",         # "ignore" | "allow" | "forbid"
    str_strip_whitespace = True,             # strip auto toutes les str
    str_to_lower         = True,             # force les str en minuscules
    populate_by_name     = True,             # accepte nom Python ET alias
    alias_generator      = to_camel,         # génère les alias auto
    validate_default     = True,             # valide les valeurs par défaut
    validate_assignment  = True,             # valide à chaque réassignation
    revalidate_instances = "always",         # revalide les instances passées
    from_attributes      = True,             # accepte les objets ORM
)
```

## Valeurs de extra

|Valeur|Comportement|
|---|---|
|`"ignore"` (défaut)|champs inconnus acceptés mais ignorés|
|`"allow"`|champs inconnus stockés dans `model.model_extra`|
|`"forbid"`|champs inconnus → `ValidationError`|

## frozen=True — immuabilité et hashabilité

```python
class Point(BaseModel):
    model_config = ConfigDict(frozen=True)
    x: float
    y: float

p = Point(x=1.0, y=2.0)
p.x = 3.0    # ❌ ValidationError: Instance is frozen
hash(p)      # ✅ hashable → utilisable comme clé de dict ou dans un set
```

## validate_assignment — validation live

```python
class User(BaseModel):
    model_config = ConfigDict(validate_assignment=True)
    age: int

u = User(age=30)
u.age = "trente"  # ❌ ValidationError même après la création
u.age = 31        # ✅
```

> [!tip] strict par champ uniquement
> 
> ```python
> from typing import Annotated
> from pydantic import Strict
> age: Annotated[int, Strict()]  # strict seulement sur ce champ
> ```
