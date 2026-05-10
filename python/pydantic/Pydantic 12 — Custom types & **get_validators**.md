#python #pydantic #custom-types #annotated #validator

## Approche 1 — Annotated + AfterValidator / BeforeValidator

```python
from typing import Annotated
from pydantic import AfterValidator, BeforeValidator, BaseModel

def to_upper(v: str) -> str:
    return v.upper()

def must_be_positive(v: int) -> int:
    if v <= 0:
        raise ValueError("Doit être positif")
    return v

UpperStr    = Annotated[str, AfterValidator(to_upper)]
PositiveInt = Annotated[int, AfterValidator(must_be_positive)]

class Product(BaseModel):
    code:  UpperStr
    stock: PositiveInt

Product(code="abc123", stock=10).code   # "ABC123"
```

## Approche 2 — **get_pydantic_core_schema** (v2, performant)

```python
from pydantic import GetCoreSchemaHandler
from pydantic_core import core_schema
from typing import Any

class HexColor:
    """Type custom : couleur CSS hexadécimale."""

    def __init__(self, value: str):
        if not value.startswith("#") or len(value) not in (4, 7):
            raise ValueError(f"Couleur hex invalide : {value}")
        self.value = value.upper()

    @classmethod
    def __get_pydantic_core_schema__(
        cls, source_type: Any, handler: GetCoreSchemaHandler
    ) -> core_schema.CoreSchema:
        return core_schema.no_info_plain_validator_function(
            cls._validate,
            serialization=core_schema.to_string_ser_schema(),
        )

    @classmethod
    def _validate(cls, v: Any) -> "HexColor":
        if isinstance(v, cls):
            return v
        return cls(str(v))

    def __str__(self):
        return self.value

class Theme(BaseModel):
    primary:   HexColor
    secondary: HexColor = HexColor("#FFFFFF")

t = Theme(primary="#ff5733")
print(t.primary)    # "#FF5733"
t.model_dump()      # {"primary": "#FF5733", "secondary": "#FFFFFF"}
```

## Approche 3 — NewType (sémantique uniquement)

```python
from typing import NewType

UserId  = NewType("UserId",  int)
OrderId = NewType("OrderId", int)

class Comment(BaseModel):
    author_id: UserId    # reste un int — lisibilité du code
    order_id:  OrderId
```

> [!tip] Quelle approche choisir ? — Logique simple → `Annotated + AfterValidator` — Type riche avec comportement → `__get_pydantic_core_schema__` — Sémantique uniquement → `NewType`
