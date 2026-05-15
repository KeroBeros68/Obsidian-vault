#python #decorateurs #fastapi #pydantic #dataclass #frameworks

## FastAPI — décorateurs de routes
```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    age:  int

@app.get("/users")             # GET /users
def list_users() -> list[User]:
    return []

@app.post("/users")            # POST /users
def create_user(user: User) -> User:
    return user

@app.get("/users/{user_id}")   # paramètre de path
def get_user(user_id: int) -> User:
    ...

@app.delete("/users/{user_id}", status_code=204)
def delete_user(user_id: int) -> None:
    ...
```

## FastAPI — décorateurs de dépendances
```python
from fastapi import Depends, HTTPException
from typing import Annotated

def get_current_user(token: str) -> User:
    ...   # logique d'auth

def require_admin(user: Annotated[User, Depends(get_current_user)]) -> User:
    if user.role != "admin":
        raise HTTPException(status_code=403)
    return user

@app.get("/admin")
def admin_panel(user: Annotated[User, Depends(require_admin)]) -> dict:
    return {"admin": user.name}
```

## Pydantic — décorateurs de validation
```python
from pydantic import BaseModel, field_validator, model_validator, computed_field

class Order(BaseModel):
    product: str
    qty:     int
    price:   float

    @field_validator("qty")
    @classmethod
    def qty_positive(cls, v: int) -> int:
        if v <= 0:
            raise ValueError("qty doit être > 0")
        return v

    @model_validator(mode="after")
    def check_total(self) -> "Order":
        if self.qty * self.price > 10_000:
            raise ValueError("Commande > 10 000€ nécessite validation manuelle")
        return self

    @computed_field
    @property
    def total(self) -> float:
        return round(self.qty * self.price, 2)
```

## dataclasses — décorateurs de classes
```python
from dataclasses import dataclass, field, fields, asdict

@dataclass
class Point:
    x: float
    y: float

@dataclass(frozen=True)          # immuable, hashable
class Config:
    host: str = "localhost"
    port: int  = 5432

@dataclass(order=True)           # __lt__, __le__, __gt__, __ge__
class Version:
    major: int = field(compare=True)
    minor: int = field(compare=True)
    label: str = field(default="", compare=False)

# Introspection
fields(Version)
asdict(Version(1, 2, "beta"))   # {"major": 1, "minor": 2, "label": "beta"}
```

## __post_init__ avec @dataclass
```python
from dataclasses import dataclass

@dataclass
class Circle:
    radius: float

    def __post_init__(self) -> None:
        if self.radius < 0:
            raise ValueError("Le rayon doit être positif")

Circle(-1)   # ❌ ValueError
Circle(5)    # ✅
```

## Combiner dataclass et Pydantic
```python
from pydantic.dataclasses import dataclass   # remplace le dataclass standard

@dataclass                       # validation Pydantic + syntaxe dataclass
class User:
    name: str
    age:  int

User(name="Alice", age="30")     # ✅ "30" coercé en 30
User(name="Alice", age="trente") # ❌ ValidationError
```

> [!tip] pydantic.dataclasses.dataclass vs BaseModel
> `@dataclass` Pydantic = syntaxe dataclass + validation Pydantic.
> `BaseModel` = plus de fonctionnalités (model_dump, model_validate, alias...).
> Utiliser `BaseModel` pour les APIs, `@dataclass` Pydantic pour les structures de données internes.
