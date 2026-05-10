#python #pydantic #optional #default #champs

## Champ obligatoire vs facultatif

```python
from pydantic import BaseModel, Field
from typing import Optional

class Product(BaseModel):
    name:        str            # ← obligatoire
    price:       float          # ← obligatoire
    stock:       int   = 0      # ← facultatif, défaut 0
    description: Optional[str] = None  # ← facultatif, peut être None
```

## Syntaxes équivalentes (Python 3.10+)

```python
description: Optional[str] = None
description: str | None    = None   # identique
```

## Valeur par défaut dynamique — default_factory

```python
from datetime import datetime

class Event(BaseModel):
    name:       str
    items:      list[str] = Field(default_factory=list)
    created_at: datetime  = Field(default_factory=datetime.utcnow)
    metadata:   dict      = Field(default_factory=dict)
```

> [!warning] Ne jamais mettre une liste ou un dict comme défaut direct
> 
> ```python
> items: list = []                            # ❌ partagé entre instances !
> items: list = Field(default_factory=list)   # ✅
> ```

## Tableau récapitulatif

|Syntaxe|Obligatoire|Peut être None|
|---|---|---|
|`name: str`|✅ oui|❌ non|
|`stock: int = 0`|❌ non|❌ non|
|`desc: Optional[str] = None`|❌ non|✅ oui|
|`desc: str = Field(...)`|✅ oui (Field explicite)|❌ non|

## model_fields_set — savoir ce qui a été fourni

```python
user = User(name="Alice", age=30)
print(user.model_fields_set)   # {"name", "age"}

# Cas d'usage : PATCH REST — ne mettre à jour que ce qui a été fourni
def patch_user(user_id: int, patch: UserPatch):
    updates = patch.model_dump(exclude_unset=True)
    db.update(user_id, updates)
```
