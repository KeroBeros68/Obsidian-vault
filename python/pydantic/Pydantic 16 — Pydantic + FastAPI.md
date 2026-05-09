#pydantic #fastapi #api #rest #openapi

## Body — valider le corps d'une requête

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class UserCreate(BaseModel):
    username: str = Field(min_length=3)
    email:    str
    age:      int = Field(ge=18)

@app.post("/users", response_model=UserOut, status_code=201)
def create_user(user: UserCreate):
    return db.create(user.model_dump())
```

## Response model — filtrer la sortie

```python
class UserOut(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id:       int
    username: str
    email:    str

@app.get("/users/{id}", response_model=UserOut)
def get_user(id: int):
    return db.get(id)
```

## Pattern Input / Output / Update

```python
class UserBase(BaseModel):
    username: str
    email:    str

class UserCreate(UserBase):
    password: str

class UserUpdate(BaseModel):
    username: str | None = None
    email:    str | None = None

class UserOut(UserBase):
    model_config = ConfigDict(from_attributes=True)
    id:         int
    created_at: datetime
```

## Query params et Path params avec validation

```python
from fastapi import Query, Path
from typing import Annotated

@app.get("/users")
def list_users(
    page:   Annotated[int, Query(ge=1)] = 1,
    limit:  Annotated[int, Query(ge=1, le=100)] = 20,
    search: Annotated[str | None, Query(max_length=100)] = None,
):
    ...

@app.get("/users/{user_id}")
def get_user(user_id: Annotated[int, Path(ge=1)]):
    ...
```

## response_model options

```python
@app.get("/users/{id}",
    response_model=UserOut,
    response_model_exclude={"email"},
    response_model_exclude_none=True,
)
def get_user(id: int): ...
```

> [!tip] Séparer Input / Output Toujours au moins 2 modèles par ressource REST : `UserCreate` (avec password) et `UserOut` (sans password, avec id). → évite les fuites de données sensibles.

> [!warning] ValidationError dans FastAPI FastAPI intercepte les `ValidationError` → `422 Unprocessable Entity` automatiquement. Pas besoin de try/except dans les routes pour la validation des corps de requête.
