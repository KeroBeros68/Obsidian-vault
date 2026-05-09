#pydantic #nested #listes #composition

## Modèle imbriqué

```python
from pydantic import BaseModel

class Address(BaseModel):
    street:   str
    city:     str
    zip_code: str

class Customer(BaseModel):
    name:    str
    address: Address

# Pydantic accepte un dict Python à la place du modèle
c = Customer(
    name="Alice",
    address={"street": "1 rue de la Paix", "city": "Paris", "zip_code": "75001"}
)
print(c.address.city)  # "Paris"
```

## Listes et dicts de modèles

```python
class Order(BaseModel):
    product:  str
    quantity: int

class Invoice(BaseModel):
    client:   str
    orders:   list[Order]
    services: dict[str, Address]   # clé str → valeur Address
```

## Modèles récursifs

```python
from __future__ import annotations
from typing import Optional

class Category(BaseModel):
    name:     str
    children: list[Category] = []   # arbre récursif

root = Category(name="Tech", children=[
    Category(name="Python", children=[Category(name="Pydantic")])
])
```

## Messages d'erreur précis

```python
# Si orders[1].quantity = "deux" → ValidationError :
# orders → 1 → quantity : Input should be a valid integer
# Le chemin complet est dans e.errors()[i]["loc"]
```

> [!tip] Parsing JSON complet en une ligne
> 
> ```python
> invoice = Invoice.model_validate_json(api_json_response)
> # valide toute la hiérarchie imbriquée en une seule opération
> ```
