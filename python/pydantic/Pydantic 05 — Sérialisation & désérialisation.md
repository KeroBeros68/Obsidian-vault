#pydantic #serialisation #json #model_dump #model_validate

## Désérialiser — créer depuis des données

```python
user = User(name="Alice", age=30)                            # kwargs
user = User.model_validate({"name": "Alice", "age": 30})     # dict
user = User.model_validate_json('{"name":"Alice","age":30}') # JSON string
user = User.model_validate(orm_obj, from_attributes=True)    # objet ORM
```

## Sérialiser — exporter

```python
user.model_dump()                  # → dict Python
user.model_dump_json()             # → JSON string
user.model_dump_json(indent=2)     # → JSON indenté
```

## Filtrer les champs à l'export

```python
user.model_dump(exclude={"password", "secret"})
user.model_dump(include={"name", "email"})
user.model_dump(exclude_none=True)       # retire les None
user.model_dump(exclude_defaults=True)   # retire les valeurs par défaut
user.model_dump(exclude_unset=True)      # retire ce qui n'a pas été fourni
user.model_dump(by_alias=True)           # utilise les alias en sortie
```

## @field_serializer — sérialisation custom d'un champ

```python
from pydantic import field_serializer
from datetime import datetime

class Event(BaseModel):
    name:       str
    created_at: datetime

    @field_serializer("created_at")
    def serialize_dt(self, v: datetime) -> str:
        return v.strftime("%d/%m/%Y %H:%M")

Event(name="Conf", created_at=datetime(2024, 6, 15, 9, 0)).model_dump()
# {"name": "Conf", "created_at": "15/06/2024 09:00"}
```

## @model_serializer — sérialisation complète custom

```python
from pydantic import model_serializer

class User(BaseModel):
    first: str
    last:  str

    @model_serializer
    def to_dict(self) -> dict:
        return {"full_name": f"{self.first} {self.last}"}

User(first="Alice", last="Martin").model_dump()
# {"full_name": "Alice Martin"}
```

## Tableau comparatif v1 → v2

|Pydantic v1|Pydantic v2|
|---|---|
|`.dict()`|`.model_dump()`|
|`.json()`|`.model_dump_json()`|
|`.parse_obj(d)`|`.model_validate(d)`|
|`.parse_raw(s)`|`.model_validate_json(s)`|
|`.schema()`|`.model_json_schema()`|
|`class Config:`|`model_config = ConfigDict(...)`|
|`@validator`|`@field_validator`|
|`@root_validator`|`@model_validator`|

> [!tip] exclude_unset pour les PATCH REST
> 
> ```python
> patch = UserPatch.model_validate({"name": "Alice"})
> patch.model_dump(exclude_unset=True)  # {"name": "Alice"} seulement
> ```
