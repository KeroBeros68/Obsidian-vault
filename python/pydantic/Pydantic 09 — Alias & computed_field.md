#pydantic #alias #computed_field #serialisation

## Alias simple

```python
from pydantic import BaseModel, Field, ConfigDict

class Product(BaseModel):
    model_config = ConfigDict(populate_by_name=True)
    name:     str   = Field(alias="product_name")
    price_ht: float = Field(alias="priceHT")

p = Product.model_validate({"product_name": "Clavier", "priceHT": 49.99})
p.model_dump(by_alias=True)
# {"product_name": "Clavier", "priceHT": 49.99}
```

## alias_generator — conversion automatique

```python
from pydantic.alias_generators import to_camel

class ApiModel(BaseModel):
    model_config = ConfigDict(alias_generator=to_camel, populate_by_name=True)
    first_name: str   # → alias "firstName" en JSON
    last_name:  str   # → alias "lastName" en JSON
```

## validation_alias vs serialization_alias

```python
class User(BaseModel):
    name: str = Field(
        validation_alias="full_name",    # lu depuis "full_name" en entrée
        serialization_alias="userName",  # exporté sous "userName" en sortie
    )

User.model_validate({"full_name": "Alice"}).model_dump(by_alias=True)
# {"userName": "Alice"}
```

## AliasPath & AliasChoices — alias complexes

```python
from pydantic import AliasPath, AliasChoices

class User(BaseModel):
    # Extraire depuis un chemin imbriqué
    city: str = Field(validation_alias=AliasPath("address", "city"))

    # Accepter plusieurs noms alternatifs
    name: str = Field(validation_alias=AliasChoices("full_name", "name", "username"))

User.model_validate({"address": {"city": "Paris"}, "full_name": "Alice"})
User.model_validate({"address": {"city": "Lyon"},  "username": "Bob"})
```

## computed_field

```python
from pydantic import computed_field

class Product(BaseModel):
    price_ht: float
    tva:      float = 0.20

    @computed_field
    @property
    def price_ttc(self) -> float:
        return round(self.price_ht * (1 + self.tva), 2)

    @computed_field(repr=False)
    @property
    def tva_amount(self) -> float:
        return round(self.price_ht * self.tva, 2)

Product(price_ht=100.0).model_dump()
# {"price_ht": 100.0, "tva": 0.2, "price_ttc": 120.0, "tva_amount": 20.0}
```

> [!tip] populate_by_name=True — toujours l'activer avec des alias Sans lui, passer le nom Python en entrée lève une `ValidationError`. Obligatoire pour les tests unitaires.
