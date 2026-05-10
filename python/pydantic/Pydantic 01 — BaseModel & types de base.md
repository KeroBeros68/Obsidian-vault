#python #pydantic #basemodel #validation #bases

## Import

```python
from pydantic import BaseModel
import pydantic
print(pydantic.__version__)  # vérifier qu'on est en v2
```

## Définir un modèle

```python
from pydantic import BaseModel

class User(BaseModel):
    name:  str
    age:   int
    email: str
```

## Créer une instance

```python
# Création valide
user = User(name="Alice", age=30, email="alice@example.com")
print(user.name)   # "Alice"
print(user.age)    # 30

# Depuis un dictionnaire
data = {"name": "Bob", "age": 25, "email": "bob@x.com"}
user = User(**data)
user = User.model_validate(data)   # forme préférée

# Création invalide → ValidationError
User(name="Bob", age="pas un int", email="x")
```

## Types supportés nativement

|Type Python|Comportement|
|---|---|
|`str`|accepté tel quel|
|`int` / `float`|coercition depuis str compatible|
|`bool`|`"true"` / `1` / `"yes"` → `True`|
|`list` / `List[T]`|valide chaque élément|
|`dict` / `Dict[K,V]`|valide clés et valeurs|
|`datetime`|`"2024-01-15T10:00:00"` → objet `datetime`|
|`date`|`"2024-01-15"` → objet `date`|
|`UUID`|string UUID → objet `UUID`|
|`Decimal`|string ou float → `Decimal`|
|`Path`|string → objet `Path`|
|`bytes`|string base64 → `bytes`|

## Coercition automatique

```python
User(name="Alice", age="30", email="x@x.com")
# age="30" (str) → 30 (int) ✅ conversion silencieuse

User(name="Alice", age="trente", email="x@x.com")
# ❌ "trente" ne peut pas devenir un int → ValidationError
```

## Inspecter un modèle

```python
user.name               # attribut Python classique
user.model_fields       # dict de FieldInfo par nom de champ
user.model_fields_set   # set des champs passés à la création
user.model_extra        # champs inconnus si extra="allow"
User.model_json_schema()   # JSON Schema complet
```

## model_copy — copier avec modifications

```python
user2 = user.model_copy(update={"age": 31})
# ✅ crée une nouvelle instance — ne modifie pas l'original
# Indispensable avec frozen=True
```

> [!tip] Coercition vs strict Par défaut Pydantic **coerce** les types. Pour désactiver : `model_config = ConfigDict(strict=True)`. Voir [[Pydantic 08 — model_config & ConfigDict]].
