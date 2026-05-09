#pydantic #pièges #erreurs #debugging

## 🪤 Piège 1 — Oublier @classmethod sur @field_validator

```python
@field_validator("username")
def check(cls, v): ...        # ❌ PydanticUserError

@field_validator("username")
@classmethod
def check(cls, v): ...        # ✅
```

## 🪤 Piège 2 — Oublier return self dans @model_validator(mode="after")

```python
@model_validator(mode="after")
def check(self):
    if self.end <= self.start:
        raise ValueError("...")
    # ❌ oubli de return self → modèle vaut None silencieusement

@model_validator(mode="after")
def check(self):
    if self.end <= self.start:
        raise ValueError("...")
    return self   # ✅
```

## 🪤 Piège 3 — strict=True casse toute la coercition

```python
M(age="30")    # ❌ strict interdit str→int
M(age=True)    # ❌ strict interdit bool→int aussi !
M(age=30)      # ✅
```

## 🪤 Piège 4 — Alias sans populate_by_name

```python
class M(BaseModel):
    name: str = Field(alias="full_name")

M(name="Alice")       # ❌ "full_name" attendu
M(full_name="Alice")  # ✅

class M(BaseModel):
    model_config = ConfigDict(populate_by_name=True)
    name: str = Field(alias="full_name")

M(name="Alice")       # ✅ les deux fonctionnent
```

## 🪤 Piège 5 — extra="ignore" masque les fautes de frappe

```python
User(username="Alice", usernmae="Bob")  # ❌ typo silencieuse !
# ✅ Utiliser extra="forbid" en développement
```

## 🪤 Piège 6 — default mutable sans default_factory

```python
items: list = []                            # ❌ partagé entre instances !
items: list = Field(default_factory=list)   # ✅
```

## 🪤 Piège 7 — Modifier un objet frozen

```python
c.host = "prod"                              # ❌ ValidationError
c2 = c.model_copy(update={"host": "prod"})  # ✅ nouvelle instance
```

## 🪤 Piège 8 — Confondre v1 et v2

```python
user.dict()           # ❌ AttributeError en v2
user.model_dump()     # ✅
User.parse_obj(d)     # ❌
User.model_validate(d)  # ✅
```

## 🪤 Piège 9 — model_construct sans validation

```python
User.model_construct(age=-999)  # ❌ aucune validation !
# À réserver aux données internes de confiance uniquement
```

## 🪤 Piège 10 — mode="before" sans @classmethod sur @model_validator

```python
@model_validator(mode="before")
def check(cls, data): ...       # ❌

@model_validator(mode="before")
@classmethod
def check(cls, data): ...       # ✅
# ≠ mode="after" qui est une méthode d'instance
```

## 🪤 Piège 11 — Lazy loading ORM avec session fermée

```python
user_orm = db.query(UserORM).first()
db.close()
UserOut.model_validate(user_orm)   # ❌ DetachedInstanceError sur les relations

# ✅ Charger en eager avant de fermer
user_orm = db.query(UserORM).options(joinedload(UserORM.posts)).first()
```

## 🪤 Piège 12 — Oublier by_alias=True à l'export

```python
u.model_dump()               # {"name": "Alice"}     ← alias ignoré
u.model_dump(by_alias=True)  # {"userName": "Alice"}  ✅
```

## 🪤 Piège 13 — Settings lus à chaque appel sans cache

```python
def get_settings():
    return Settings()   # ❌ relit le .env à chaque requête

@lru_cache()
def get_settings():
    return Settings()   # ✅ lu une seule fois au démarrage
```

## 🪤 Piège 14 — info.data dans field_validator incomplet

```python
class M(BaseModel):
    a: str
    b: str
    c: str

    @field_validator("b")
    @classmethod
    def check(cls, v, info: ValidationInfo):
        # info.data contient SEULEMENT {"a": ...}
        # "c" n'est pas encore validé → absent de info.data !
        return v
```

## Récapitulatif rapide

|Piège|Solution|
|---|---|
|`@field_validator` sans `@classmethod`|Toujours les deux ensemble|
|Oublier `return self` en `mode="after"`|Modèle vaut None sinon|
|Alias sans `populate_by_name`|Ajouter dans ConfigDict|
|`extra="ignore"` masque les typos|`extra="forbid"` en développement|
|`default=[]` mutable|`Field(default_factory=list)`|
|Modifier un `frozen`|`model_copy(update={...})`|
|Méthodes v1 en v2|`model_dump()`, `model_validate()`|
|`model_construct` sur données externes|Uniquement données de confiance|
|Lazy loading ORM|`joinedload()` / `selectinload()` avant session close|
|Oublier `by_alias=True`|`model_dump(by_alias=True)`|
|Settings sans `@lru_cache`|Toujours cacher le `get_settings()`|
|`info.data` incomplet dans validator|Ordre déclaration = ordre validation|
