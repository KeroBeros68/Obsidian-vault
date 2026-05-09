#pydantic #sqlalchemy #orm #database #schema

## Pattern de base — schéma séparé du modèle ORM

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import DeclarativeBase
from pydantic import BaseModel, ConfigDict

class Base(DeclarativeBase):
    pass

class UserORM(Base):
    __tablename__ = "users"
    id:       int = Column(Integer, primary_key=True)
    username: str = Column(String)
    email:    str = Column(String)
    password: str = Column(String)

# Schémas Pydantic séparés
class UserBase(BaseModel):
    username: str
    email:    str

class UserCreate(UserBase):
    password: str                    # entrée : avec password

class UserOut(UserBase):
    model_config = ConfigDict(from_attributes=True)
    id: int                          # sortie : + id, sans password
```

## from_attributes=True — lire les attributs ORM

```python
user_orm = db.query(UserORM).filter_by(id=1).first()
user_out = UserOut.model_validate(user_orm)
# ✅ Pydantic lit user_orm.id, user_orm.username, user_orm.email
```

## Relations ORM — attention au lazy loading

```python
class PostOut(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id:    int
    title: str
    user:  UserOut   # ← déclenchera un lazy load si non eager-loaded !

# ✅ Charger en eager avant de fermer la session
from sqlalchemy.orm import joinedload
post_orm = db.query(PostORM).options(joinedload(PostORM.user)).first()
```

## Schéma d'héritage complet

```python
class ItemBase(BaseModel):
    name:  str
    price: float

class ItemCreate(ItemBase):
    pass

class ItemUpdate(BaseModel):
    name:  str | None = None   # tout optionnel pour PATCH
    price: float | None = None

class ItemOut(ItemBase):
    model_config = ConfigDict(from_attributes=True)
    id:         int
    created_at: datetime
```

## CRUD FastAPI + SQLAlchemy

```python
@app.post("/items", response_model=ItemOut)
def create_item(item: ItemCreate, db: Session = Depends(get_db)):
    db_item = ItemORM(**item.model_dump())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return ItemOut.model_validate(db_item)

@app.patch("/items/{id}", response_model=ItemOut)
def update_item(id: int, patch: ItemUpdate, db: Session = Depends(get_db)):
    db_item = db.get(ItemORM, id)
    for k, v in patch.model_dump(exclude_unset=True).items():
        setattr(db_item, k, v)
    db.commit()
    return ItemOut.model_validate(db_item)
```

> [!warning] Lazy loading et session fermée Si la session SQLAlchemy est fermée quand Pydantic accède à une relation → `DetachedInstanceError`. Toujours charger en eager avec `joinedload()` ou `selectinload()`.
