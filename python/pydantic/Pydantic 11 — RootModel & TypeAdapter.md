#pydantic #rootmodel #typeadapter #validation

## RootModel — valider un type sans champs nommés

```python
from pydantic import RootModel

class Tags(RootModel[list[str]]):
    pass

tags = Tags.model_validate(["python", "pydantic"])
print(tags.root)         # ["python", "pydantic"]
tags.model_dump()        # ["python", "pydantic"]
tags.model_dump_json()   # '["python","pydantic"]'

class ScoreMap(RootModel[dict[str, float]]):
    pass

scores = ScoreMap.model_validate({"alice": 9.5, "bob": 8.0})
print(scores.root["alice"])  # 9.5
```

## RootModel avec méthodes custom

```python
class UserList(RootModel[list[User]]):
    def __iter__(self):
        return iter(self.root)

    def __len__(self):
        return len(self.root)

    def __getitem__(self, idx):
        return self.root[idx]

    def find_by_name(self, name: str) -> User | None:
        return next((u for u in self.root if u.name == name), None)

users = UserList.model_validate([{"name": "Alice", "age": 30}])
for user in users:
    print(user.name)
```

## TypeAdapter — valider sans créer de classe

```python
from pydantic import TypeAdapter
from typing import Annotated
from pydantic import Field

ta_int  = TypeAdapter(int)
ta_list = TypeAdapter(list[str])

ta_int.validate_python("42")               # → 42
ta_list.validate_python(["a", "b"])        # → ["a", "b"]
ta_list.validate_json('["x","y"]')         # → ["x", "y"]

# Avec Annotated
PositiveInt = Annotated[int, Field(gt=0)]
ta = TypeAdapter(PositiveInt)
ta.validate_python(5)    # ✅
ta.validate_python(-1)   # ❌ ValidationError

# Sérialiser
ta_list.dump_python(["a", "b"])   # → ["a", "b"]
ta_list.dump_json(["a", "b"])     # → b'["a","b"]'

# JSON Schema
ta_list.json_schema()
```

> [!tip] TypeAdapter vs BaseModel `TypeAdapter` est idéal pour les types primitifs, les listes, les dicts, ou les `Annotated` réutilisables — sans classe dédiée. Très utile dans les pipelines de données et les fonctions utilitaires.
