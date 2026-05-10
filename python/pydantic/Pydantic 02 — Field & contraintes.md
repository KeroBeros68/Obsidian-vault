#python #pydantic #field #contraintes #validation

## Field — ajouter des contraintes

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    username: str   = Field(min_length=3, max_length=20)
    age:      int   = Field(ge=0, le=120)
    score:    float = Field(default=0.0, ge=0.0, le=100.0)
    bio:      str   = Field(default="", description="Biographie publique")
```

## Contraintes numériques

|Paramètre|Signification|Exemple|
|---|---|---|
|`ge=n`|≥ n (greater or equal)|`ge=0` → 0 inclus|
|`gt=n`|> n (strictly greater)|`gt=0` → 0 exclu|
|`le=n`|≤ n (less or equal)|`le=100`|
|`lt=n`|< n (strictly less)|`lt=100` → 99 max|
|`multiple_of=n`|multiple de n|`multiple_of=5`|

## Contraintes texte

|Paramètre|Signification|
|---|---|
|`min_length=n`|longueur minimale|
|`max_length=n`|longueur maximale|
|`pattern="regex"`|doit matcher le regex|

```python
class Product(BaseModel):
    ref:  str = Field(pattern=r"^[A-Z]{2}\d{4}$")  # ex: AB1234
    slug: str = Field(pattern=r"^[a-z0-9-]+$")
```

## Contraintes de collection

```python
class Survey(BaseModel):
    tags: list[str] = Field(min_length=1, max_length=10)
    # min_length / max_length s'appliquent au nb d'éléments
```

## Metadata et options d'affichage

```python
class Product(BaseModel):
    name:     str  = Field(description="Nom commercial du produit")
    price:    float = Field(gt=0, examples=[9.99, 49.99])
    password: str  = Field(repr=False)    # masqué dans print()
    internal: str  = Field(exclude=True)  # exclu de model_dump()
```

> [!tip] Field vs annotation seule `age: int` → valide tout int `age: int = Field(ge=0, le=120)` → valide + contraint `repr=False` est utile pour les champs sensibles (password, token).
