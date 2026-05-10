#python #typing #annotated #metadata #pydantic

## Annotated — attacher des métadonnées à un type


```python
from typing import Annotated

# Annotated[type, metadata1, metadata2, ...]
# Le type reste le premier argument — le reste est ignoré par Python
# mais utilisé par les librairies tierces (Pydantic, FastAPI...)

x: Annotated[int, "doit être positif"]   # Python ignore le str
```

## Avec Pydantic — contraintes réutilisables


```python
from typing import Annotated
from pydantic import BaseModel, Field

# Définir les types annotés une fois
Age         = Annotated[int,   Field(ge=0, le=120)]
Email       = Annotated[str,   Field(pattern=r".+@.+\..+")]
PositiveInt = Annotated[int,   Field(gt=0)]
ShortStr    = Annotated[str,   Field(min_length=1, max_length=100)]

# Réutiliser dans plusieurs modèles
class User(BaseModel):
    name:  ShortStr
    age:   Age
    email: Email

class Employee(BaseModel):
    name:   ShortStr
    salary: PositiveInt
    age:    Age
```

## Avec FastAPI — validation des paramètres


```python
from fastapi import Query, Path
from typing import Annotated

@app.get("/users")
def list_users(
    page:  Annotated[int, Query(ge=1, description="Numéro de page")] = 1,
    limit: Annotated[int, Query(ge=1, le=100)] = 20,
):
    ...
```

## Annotated comme documentation machine-readable


```python
from typing import Annotated

# Metadata custom — les librairies lisent ce qu'elles connaissent
class Positive: pass
class NonEmpty: pass

Score  = Annotated[float, Positive()]
Name   = Annotated[str,   NonEmpty()]

# beartype, Pydantic, typer... lisent ces métadonnées à leur manière
```

## get_type_hints — récupérer les annotations résolues


```python
import typing

class Config:
    host: str
    port: Annotated[int, "TCP port"]

hints = typing.get_type_hints(Config, include_extras=True)
# {"host": str, "port": Annotated[int, "TCP port"]}

hints_plain = typing.get_type_hints(Config)
# {"host": str, "port": int}  ← sans les extras
```

> [!tip] Annotated est transparent pour Python Le type réel (premier argument) est celui utilisé pour les vérifications `isinstance`, `issubclass`, etc. Les métadonnées sont invisibles au runtime Python — elles existent uniquement pour les outils et frameworks.
